# Circle — a case study in designing for audience isolation

A one-way family notice board for families that span more than one household. Working name. No production code exists: this is a product and architecture design, taken to the point where it could be built, and written up here because the design is the interesting part.

The spec, backlog and architecture live in a private repo. This one is the write-up.

---

## The problem

A parent in a family where both adults have divorced parents needs to keep four sides plus extended family updated on the kids. The obvious answer is a group chat, and it works for about a week.

It works for the first job, which is telling everyone once instead of being the switchboard. It fails on the second, which is controlling what each side sees. Grandparents on one side see what grandparents on the other side gave, said, and were invited to. That is where the comparison and the score-keeping start, and it is why a birthday party turns into a conflict.

The first job is served adequately by a group chat. The second is served by nothing on the market.

## Why the category cannot fix this

I looked at more than thirty products across private family sharing and co-parenting, and asked each one the same two questions: can a subscriber see another subscriber's reply, and can a subscriber tell how many other people are in the room.

Every one of them is built on a shared-room primitive. An invited relative is a member of a space, and membership is visible. Co-parenting apps get closest, because hostile households are their whole reason to exist, but they solve it between two parents, not across an extended family of eight people in four households.

This is not a feature gap. Adding per-person visibility to a shared room means rebuilding the room, which is why nobody has done it.

## The decision that cannot be retrofitted

The primitive is audience isolation: every subscriber has a private, one-to-one relationship with the owner, and no lateral visibility to anyone else. Posts, photos, profiles and calendar are content flowing through that primitive.

In the data model that comes down to one thing. `Thread` is keyed on `(circle_id, subscriber_id)`, not on `post_id`. **A reply belongs to the relationship, not to the post.** Claims, RSVPs and reactions sit on the same shape.

Build replies as children of a post, the way every competitor does, and you have built a group chat with hidden UI. Every other mistake here is recoverable: audience groups wrong is a query fix, notifications wrong is a job fix. This one is a rewrite, so it is the decision that had to be right before anything else was designed.

The rule that falls out of it: there is no join path from one subscriber to another. If a query can traverse subscriber to object to subscriber, the schema is wrong.

## Sixteen ways isolation leaks without a rule being broken

The hard part is not the permission check. It is everything that discloses another person's existence while every rule is being followed correctly. I catalogued sixteen, and each one has a test.

A few that are easy to miss:

- **Reply counts.** "3 people replied" tells a subscriber there are others. So no counts, no aggregates, no "someone else is viewing" anywhere in the product.
- **Sequential IDs.** They let a subscriber infer volume and probe for neighbours. An item id is a probe target and a claim id is a person, so UUID v4 only, and specifically not v7, because v7 embeds a timestamp.
- **Timing.** A post edited or deleted right after a reply implies who replied.
- **Email headers.** A `to:` or `cc:` with more than one recipient destroys the whole model in a single send. One send per subscriber, no BCC, no list headers, and a per-subscriber opaque reply-to token so replies cannot land in a shared inbox.
- **Link preview crawlers.** A messaging platform's preview bot fetches the link and burns a single-use token before the human has clicked it. Tokens are therefore never consumed by a GET.
- **Backup windows.** A stated thirty-day deletion with ninety-day backups is a false claim, not a rounding error.

The full catalogue, and what each one forces in the architecture, is in [docs/visibility-model.md](docs/visibility-model.md).

One leak is accepted rather than closed. On a claimable list, a subscriber who claimed one item and can see nine others claimed knows other people exist. That is accepted, only where the owner explicitly chose it for that list, because the alternative is duplicate gifts. It is never accepted for RSVPs, where the slot is a person rather than an item.

## Making the boundary real instead of a UI convention

The server decides who sees what, at query time, on every read and every write. Client-side hiding is the failure mode that leaks, so no endpoint returns an object the caller is not entitled to, even with a flag telling the client to hide it. The calendar never returns a full event with `hidden: true` for the client to respect.

Two consequences worth naming:

**Invisible and non-existent return the same response.** A 403 for an object that exists and a 404 for one that does not discloses existence. So the key is absent from the payload entirely, error bodies are constant, and the query path is the same so the timing matches. That is what makes "a subscriber with only the gift list sees no sign that a timeline exists" true rather than decorative.

**The permission matrix is executable.** The 28-row matrix is stored as data, and generates "must not see" and "must not change" tests across the API, the rendered pages and the email output. CI fails if any matrix row or any named leak has no case. The assertion searches the whole serialised output, JSON, HTML or email, for the other subscribers' IDs, names and emails, and for any integer equal to the number of other participants.

## What the constraints forced

Everything below was decided before any code, because each one is expensive to reverse.

- **Canadian residency end to end.** Quebec's Law 25 has no size threshold and requires a privacy impact assessment before putting a Quebec resident's personal information on US infrastructure. Every vendor layer is pinned to a Canadian region rather than running the assessment or geo-excluding Quebec at launch.
- **Subscribers have no account.** No password, ever, in any subscriber flow. A single-use link, confirmed by an explicit POST, creates a revocable session scoped to one subscriber in one circle. Invite acceptance among relatives over 65 is the ceiling on the whole product, and a signup wall is where that ceiling gets low.
- **First-party telemetry only.** No third-party analytics, error tracking, session replay, tag manager, font CDN or ad SDK. Events and scrubbed errors stay in the Canadian database and metrics are SQL views. A product whose premise is that nobody else sees your children cannot ship a page that calls out to six vendors.
- **The child is a subject, never a user.** No mechanism by which a child under 13 gets an account, a login, or the ability to post. That also keeps the product outside COPPA, since the rule covers information collected from children, not information adults record about them.
- **The security sentence is fixed, verbatim.** We are not end-to-end encrypted, our servers can access content, and the published wording says so on the marketing page rather than in a policy footnote. Six words are banned outright. Overclaiming here is the specific thing the FTC took Zoom apart for.

## Reviewed like a product that already exists

I ran the design through a security review in a CISO frame against five frameworks: OWASP Top 10 2025, the OWASP Top 10 for LLM Applications, the OWASP ML Security Top 10, MITRE ATLAS, and the NIST AI Risk Management Framework. It produced 31 items, and I accepted all of them into the backlog as their own epic.

The point of doing it at design stage is that findings are cheap. Two examples of what it caught:

- One-click RSVP from an email would have been actioned by corporate mail scanners on the recipient's behalf. Every action link now lands on a confirmation page and acts only on an explicit POST.
- Nothing in the spec prevented recipient A's reply token from landing in recipient B's email, the classic fan-out bug that turns a private reply into impersonation. The fix is a fan-out test with fifty synthetic recipients on every template, plus a mail adapter whose type accepts a single recipient and which has no batch method anywhere in the codebase.

The uncomfortable finding was that the AI tooling I am building with is the least controlled surface in the whole system, and the most likely place children's data meets a model first. Details in [docs/security-review.md](docs/security-review.md).

## Scope, and the gate it has to pass

162 MUST stories for v1. A 53-story pilot cut, defined as the smallest thing that can answer whether the isolation boundary holds under real family use. A single confirmed leak in the pilot stops everything.

Before any production code, a six-week validation plan with kill criteria written in advance. The deciding question for the twelve parent interviews is "has anyone ever seen something in a family group that you wish they hadn't?" If fewer than six of twelve can produce a story within ten seconds, the pain is theoretical and the project stops.

The risk I have written down about my own work: v1 scope roughly doubled while nothing has been validated. That is recorded as a live risk, not resolved.

## Decisions I recorded as disagreements with myself

Four decisions run against my own value-to-effort ranking or risk analysis. Rather than quietly reweighting the analysis to match the decision, they are recorded in their own section with the trade-off intact.

The full free/busy calendar has five of the six worst value-to-effort ratios in the product, sits on the highest-risk surface, and no user asked for it. It is in v1 because it is the differentiator for genuinely hostile situations, and it is marked as the first thing to cut if v1 runs long.

The rest, with what was rejected and why, is in [docs/decisions.md](docs/decisions.md).

## How this was produced

I directed Claude through the whole design: as a researcher for the competitive and regulatory work, as a technical architect for the platform decisions, and in a CISO frame for the security review. Every decision has my name against it because I approved or rejected each one, and several of the interesting ones are where I overrode the recommendation.

What that bought me is scope: a thirty-product competitive sweep, a five-framework security assessment and a full architecture in the time a solo PM would normally spend on one of the three. What it did not buy me is judgment. The sixteen-leak catalogue exists because I kept asking what else discloses a person without breaking a rule, and the calendar is in v1 against the ranking because I decided the hostile-situation case was worth the cost.

More on the working method, including what I threw out, in [docs/method.md](docs/method.md).

---

## In this repo

| File | What it covers |
|---|---|
| [docs/visibility-model.md](docs/visibility-model.md) | The primitive, the data model shape, all sixteen inference leaks, and how each is answered |
| [docs/decisions.md](docs/decisions.md) | The decisions that shaped the product, each with the alternative that was rejected and why |
| [docs/security-review.md](docs/security-review.md) | The design-stage security assessment: frameworks, results, and the findings worth reading |
| [docs/method.md](docs/method.md) | How the work was produced, document ownership rules, and what I would do differently |

Names, family details and user research have been removed or anonymised throughout. The product name is a placeholder.

Josh Flippance · [joshflippance.com](https://joshflippance.com) · [LinkedIn](https://linkedin.com/in/joshflippance)
