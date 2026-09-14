# Method

How the work was produced, and the rules the documents follow.

## One fact, one place

Every document declares what it owns and what it must never contain.

| Document | Owns | Never contains |
|---|---|---|
| Evidence | Every number, price, statistic, competitor claim, legal fact | Opinions, recommendations, requirements |
| Decisions | Every judgement: framing, personas, positioning, scope, pricing, risk, go/no-go | Primary facts, requirement text |
| Spec | Every requirement, the visibility model, the data model, metrics, release criteria | Market evidence, re-argued decisions |
| Backlog | Every story, estimate, dependency, sequencing choice | Facts, rationale |
| Sources and method | Every URL, confidence rating, verification note | Anything asserted without a source |

If a figure is wrong, it is wrong in exactly one place. The cost of this discipline is real: it takes longer to write and forces cross-references instead of restating. The benefit showed up immediately, when a pricing change and a scope change each landed in one file rather than five.

One deliberate exception: the walkthrough deck restates content, because a presentation has to stand alone. It is treated as a derived artifact and regenerated rather than hand-edited.

## Working with the model

I directed Claude through the design in three distinct frames, and the framing mattered more than the prompting.

**As researcher.** A sweep of more than thirty products in the category, each asked the same two questions, with a confidence rating and verification note per claim. Unverified claims are marked unverified in the evidence file rather than quietly promoted, including the one claim that would most weaken my own white-space finding. Three market-sizing vendors are marked "do not cite" and are not cited.

**As technical architect.** Twelve architecture decisions, each presented with the alternatives and the trade-off, approved or rejected one at a time. Twenty-two lower-level technical defaults marked as proposed rather than decided, so they could be approved in one pass instead of interrupting the design. Twenty-two open questions left explicitly open rather than guessed.

**In a CISO frame.** The five-framework review, written as an adversary rather than as an author. This is the frame that produced the findings I would not have found, because it was not defending the design it was reviewing.

## What that bought, and what it did not

It bought scope. A thirty-product competitive sweep, a five-framework security assessment and a full architecture, at five to ten hours a week, is not something a solo PM gets to have.

It did not buy judgment, and the places that matter are the places I overrode the output. The calendar is in v1 against its own ranking. The absolute version of the privacy promise was narrowed because it was unkeepable. The aggregate progress mode was rejected outright rather than offered as a middle option. The sixteen-leak catalogue exists because I kept asking what else discloses a person without breaking a rule, and each answer produced another.

The failure mode I watched for: a model will happily produce a confident, complete, internally consistent document about something nobody wants. Which is why the kill criteria were written before the build sequence, and why every decision carries the alternative it beat.

## What I would do differently

**Validate before scoping.** v1 roughly doubled in size while nothing had been validated. That is recorded as a live risk in the project rather than resolved, and it is the honest lesson here: it is much easier to keep designing than to go and ask twelve people a hard question.

**Get user input earlier.** The first prospective user's written input changed the product, adding a whole job to be done that desk research had not surfaced. That was one person, several weeks in, and it reordered the backlog. It should have been week one.

**Separate proposed from decided sooner.** The convention of marking lower-level choices as proposed, with alternative and trade-off attached, arrived partway through the architecture. Before that, technical defaults were getting mixed in with product decisions and both were moving slowly.
