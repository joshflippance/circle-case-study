# Decisions

Each with the alternative that was rejected and the reason. Product and platform decisions only; the full log is longer.

## 1. The reply belongs to the relationship, not the post

**Chosen:** `Thread` keyed on `(circle_id, subscriber_id)`. A reply is never a child of a post.
**Rejected:** replies as children of a post, the way every competitor does it.
**Why:** it is the only decision that cannot be retrofitted. Audience groups wrong is a query fix. Notifications wrong is a job fix. Replies built as children of a post is a group chat with hidden UI, and unwinding it is a rewrite.

## 2. Two audience models, deliberately

**Chosen:** post and milestone audience resolved and pinned at send time. List and event audience mutable while the object is open, with a confirmation naming exactly what a newly added person will see.
**Rejected:** one audience model for everything.
**Why:** a post is a moment and its audience is pinned. A list is a standing invitation and people join it late. Forcing one model breaks one of the two use cases. The cost is two sets of tests, paid knowingly.

## 3. Per-list progress visibility, owner's explicit choice, no default

**Chosen:** owner-only or item-state, chosen at list creation, consequence shown in one plain line.
**Rejected:** (a) the absolute v0.1 promise that a subscriber never learns others exist; (b) a bare aggregate mode such as "18 of 25".
**Why:** the absolute version produces duplicate gifts, which is the thing families are trying to avoid. The aggregate is strictly worse than either option: it discloses that others exist without preventing the duplicate.

## 4. Never build a child account

**Chosen:** the child is a subject, never a user. No mechanism by which a child under 13 obtains an account, login, profile or ability to post.
**Rejected:** a limited "kid view".
**Why:** product reasons first, and a regulatory consequence that follows for free. COPPA covers information collected *from* children; information adults record *about* children sits outside the Rule. Building a kid view would move the whole product inside it.

## 5. Host in Canada, no cross-border transfer

**Chosen:** every vendor layer pinned to a Canadian region.
**Rejected:** run a Quebec privacy impact assessment and host in the US; geo-exclude Quebec at launch.
**Why:** Law 25 has no size threshold. A solo builder with zero users owes the same assessment as an enterprise before a Quebec resident's personal information touches US infrastructure. Pinning the region is cheaper than the assessment and honest about the promise being made.

## 6. Subscribers have no account and no password

**Chosen:** owners on managed auth with MFA. Subscribers get a single-use link, confirmed by an explicit POST, creating a revocable session scoped to one subscriber in one circle.
**Rejected:** everyone on managed auth; a cross-circle subscriber identity.
**Why:** invite acceptance among relatives over 65 is the ceiling on the entire product, and a password wall is where that ceiling drops. Accepted residual: a shared or found device reaches everything that subscriber can see. Recorded as a disagreement below.

## 7. Free core, charge for what costs money

**Chosen:** free core with paid tier at $59/year, priced with a deliberate path to $79.
**Rejected:** paid-only with no free tier at $39/year; per-family subscription with a generous free tier at $79 as the starting point.
**Why:** the differentiator is cheap to run and should spread. The expensive things, storage and delivery, are what get metered. It also gives the cleanest read on the one metric the growth thesis depends on, which is whether subscribers become owners.

## 8. Say precisely what the product does

**Chosen:** the published security sentence is fixed verbatim, states plainly that the product is not end-to-end encrypted and that servers can access content, and sits on the marketing page rather than in a policy. Six specific words are banned from all copy.
**Rejected:** the usual category language.
**Why:** the FTC took Zoom apart for exactly this overclaim. Getting the wording fixed before any marketing exists costs nothing; retrofitting honesty after launch costs a rewrite of every surface.

---

# Recorded disagreements

Four decisions run against the value-to-effort ranking or the risk analysis. They are kept in their own section, with the trade-off intact, so the reasoning survives the decision.

**Full free/busy calendar in v1.** Five of the six worst value-to-effort ratios in the product, on the highest-risk surface, with no user asking for it. Kept because it is the differentiator for genuinely hostile situations. Marked as the first thing to cut if v1 runs long.

**Billing in v1.** The lowest user value of anything scored, committed before the invite loop has been shown to convert. Second thing to cut.

**Named guests on an RSVP.** Creates data subjects with no account, no notice and no deletion route, entered by subscribers rather than by the owner. A subscriber can enter another person's name, possibly a child's, through a path nobody controls. The controls are a mitigation, not a solution, and it is recorded as an accepted risk rather than a solved problem.

**Long subscriber sessions with no step-up.** A shared or found device reaches everything that subscriber can see. Chosen to protect invite acceptance among the over-65 group, which is the ceiling on the product.

## One boundary crossing still open

On a milestone covering more than one child, the most permissive grant wins: a subscriber granted one child's timeline sees content about a child they were not granted. Chosen because it matches owner intent when posting about siblings. Round-two user feedback proposes reversing it to deny-beats-allow. Proposed, not applied.
