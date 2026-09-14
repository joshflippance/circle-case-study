# Design-stage security review

Run in a CISO frame against five frameworks, before any code existed. Status: accepted, with all findings taken into the backlog as their own epic.

A design-stage review has a specific limit worth stating: a "Strong" rating means the requirement is specified and testable, not that it works. Nothing here has been tested against a running system, because there isn't one.

## Frameworks and results

| Framework | Assessed | Result |
|---|---|---|
| OWASP Top 10 | 2025 | 2 Strong, 5 Partial, 3 Gap |
| OWASP Top 10 for LLM Applications | 2026 | No product exposure in v1. Build tooling: 4 of 10 apply today, 0 controlled |
| OWASP ML Security Top 10 | 2023 draft | N/A by design, and stays N/A while the no-training commitment holds |
| MITRE ATLAS | v5.6.0 | 10 techniques relevant, 0 mitigated in the spec |
| NIST AI RMF | 1.0 | Govern partial, Map gap, Measure N/A today, Manage gap |

Strong on broken access control and insecure design, which is what you would expect from a product whose entire premise is an authorization boundary. Gaps on injection, software and data integrity, and mishandling of exceptional conditions.

**31 items** in total, split between product and platform security and AI use and governance. Each carries a framework tag, an estimate, an architecture phase and a pilot flag. 22 of the 31 land in the pilot cut, roughly 47 dev-days.

## Findings worth reading

**Nothing happens because a machine opened an email.** One-click RSVP and claim-from-email directly contradicted the requirement that no action is taken without human intent. Email security scanners follow every link in every message and would have RSVP'd, claimed and unsubscribed on recipients' behalf. Every action link now lands on a confirmation page and acts only on an explicit POST. The residual is recorded honestly: some corporate mail products render and click, and CSRF protection does not solve that.

**Invisible and non-existent must return the same response.** A 403 for an object that exists and a 404 for one that does not discloses existence, which defeats the product's central claim. Implemented as: key absent from the payload entirely, constant error bodies, and the same indexed query so response timing matches.

**The fan-out bug.** Nothing in the spec prevented recipient A's reply token or content from landing in recipient B's email, which turns a private reply into impersonation. Two controls: a fan-out test with fifty synthetic recipients on every template, and a mail adapter whose type signature accepts a single recipient, with no batch method anywhere in the codebase.

**The production canary.** Two synthetic circles assert the full must-not-see set against production continuously and page on failure. A leak in production should be found by a test, not reported by a family.

**The AI feature gate.** If the product ever drafts replies, never more than one subscriber's content in a single context. The concrete failure is an AI-drafted reply to one grandparent quoting the other grandparent's message, which the owner then sends. Isolation would be broken by the feature meant to help.

## The uncomfortable finding

The product has no AI exposure in v1. The *build tooling* does, and it is the least controlled surface in the system.

Four MITRE ATLAS techniques apply to how this product is being built: prompt injection, agent tool invocation, tool credential harvesting, and data destruction through an agent's tools. Zero are controlled today. The tooling is also the most likely place children's data would first meet a model.

Recording that in the assessment is not the fix, and it is not treated as one. It is the finding with the least written against it so far, and the one I would want a reviewer to push me on.

## Where the posture is already ahead of the category

Reply keyed on the relationship rather than the post. Audience pinned at post time. Tests written before features. A named inference-leak catalogue with a case for each entry. Honest published security wording, fixed before any marketing exists. A no-training commitment that keeps an entire framework out of scope.

## Still open

How the privacy requirements stay true with email and push as first-class delivery surfaces. Who owns AI use and security decisions as the project grows past one person. One live contradiction inside the backlog: an early story still offers passwords against a requirement that says there are none, pending a platform decision.
