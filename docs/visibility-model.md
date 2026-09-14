# The visibility model

The load-bearing part of the product. Everything else is content flowing through it.

## The promise

> No subscriber learns another subscriber's identity or attribution.

That is v0.2 of the promise. v0.1 was broader: no subscriber learns another subscriber *exists*. It was narrowed deliberately, because on a claimable list, and only where the owner chooses it, a subscriber may see that items have been claimed, which implies other contributors. Narrowing a promise you cannot keep is better than keeping a promise that produces duplicate gifts.

## The primitive

Audience isolation. Every subscriber has a private, one-to-one relationship with the owner, and no lateral visibility to any other subscriber.

## What that forces in the data model

**Thread is keyed on `(circle_id, subscriber_id)`, not `post_id`.** A reply belongs to the relationship, not to the post. Claims, RSVPs and reactions take the same shape. This is the only decision in the product that cannot be retrofitted.

**There is no join path from one subscriber to another.** If a query can traverse subscriber to object to subscriber, the schema is wrong. This is the rule the schema review is actually checking.

**Post and milestone audiences are materialised at send time.** Adding someone to an audience group later must not hand them the group's history. Pinning the audience at send time makes retroactive widening structurally impossible rather than merely prohibited.

**List and event audiences are deliberately mutable while the object is open**, because a list is a standing invitation rather than a moment. Two audience models means two sets of tests, and adding someone to an open list shows a confirmation naming exactly what they will now see, including which items are already claimed.

**Claim attribution expires.** `Claim.subscriber_id` is nulled at twelve months, so an archived list keeps what was collected without keeping who gave what.

## The permission matrix

28 rows across four actors: owner, co-owner, subscriber targeted, subscriber not targeted. Values are visible, not visible, own-only, or owner's per-object choice.

Always **not visible** to any subscriber: the subscriber roster, the subscriber count, audience group names, another subscriber's reply, poll aggregates, claimant identity, claimant count, item received state, another person's RSVP, headcount, attendee list, invitee list, non-responder list, read receipts.

Owner's choice appears exactly twice: whether a list item shows as claimed or open, and whether an event shows its location.

The matrix is stored as data and generates the test suite. CI fails if any row, or any named leak below, has no case.

## The sixteen inference leaks

The ways isolation leaks with every rule being followed correctly.

| # | Leak | What it forces |
|---|---|---|
| L1 | Reply counts | No counts, aggregates or presence indicators anywhere |
| L2 | Poll results | Aggregates are owner-only; subscribers see their own answer |
| L3 | Ordering and IDs | UUID v4 only. An item id is a probe target, a claim id is a person |
| L4 | Timing | An edit or delete right after a reply implies who replied |
| L5 | Retroactive audience widening | Audience pinned at post time |
| L6 | Email and push metadata | One send per subscriber. No BCC, no list headers |
| L7 | Reply-to addresses | Per-subscriber opaque reply-to token, never a shared inbox |
| L8 | Shared invite links | No reusable invite link exists at all |
| L9 | Photo EXIF | GPS and device metadata stripped on ingest, before any fetch |
| L10 | iCal feed URLs | An unauthenticated bearer URL gives a leaked holder everything, silently, forever |
| L11 | Busy-block shape | A weekly Tuesday 5 to 6pm block is identifiable even with no title |
| L12 | Support tooling | No support view that shows one subscriber's content to another |
| L13 | Link preview crawlers | Tokens are never consumed by a GET |
| L14 | Claim state as an existence signal | Accepted, only per-list and only where the owner chose it |
| L15 | Signed media URLs in email | No photos in notification emails. A link in an inbox is a bearer token |
| L16 | Backups outside the retention promise | 30-day deletion with 90-day backups is a false claim |

Every one is mapped to a specific architectural answer, and every one has a test written before the feature it applies to.

## Why L14 is accepted

A subscriber who claimed one item and sees nine others claimed knows other people exist. Three options were on the table:

1. Hold the absolute promise. Nobody sees any claim state. Families get duplicate gifts, which is the problem they came to solve.
2. A bare aggregate: "18 of 25 claimed." This was **deliberately never built**. A count without item detail discloses that others exist *and* fails to prevent the duplicate. Worst of both.
3. Per-list, owner's explicit choice at creation, no default, with the consequence stated in one plain line.

Option 3, and the leak is accepted knowingly. It is never accepted for RSVPs, where the slot is a person rather than an item.

## The calendar

Two axes kept separate, because merging them is where every calendar product leaks: a per-subscriber access level (none, busy only, titles and locations, full), and a per-event override (default, private, shared). Availability is held as its own status.

The masked block reads exactly **"Busy"**. Not "Private", not "Unavailable", not the event's category. Terminology is not up for debate, because every variation tells you something.

## Enforcement

The server decides who sees what, at query time, on every read and every write.

- No endpoint returns an object the caller is not entitled to, even with a flag telling the client to hide it.
- The database is the authorization spine and is reached only through the server. The app presents evidence rather than asserting identity; the database resolves who that is and applies row-level security. Unset, unknown or stale evidence resolves to null, and null matches nothing.
- Subscribers read through purpose-built views with column-level access on an allow-list, and hold no write permission anywhere. Subscriber writes go through definer functions.
- Owner preview runs the subscriber path read-only, so what the owner is shown is what the subscriber is actually served rather than a simulation of it.
- Invisible and non-existent are indistinguishable: key absent, constant error bodies, matching query timing.
- A synthetic canary asserts the must-not-see set against production continuously and pages on failure. A leak in production should be detected, not reported by a family.
