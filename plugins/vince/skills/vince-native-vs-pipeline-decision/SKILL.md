---
name: vince-native-vs-pipeline-decision
description: Use whenever deciding how an M3 read or write should be implemented in a Vince Live workflow — the native API step vs. the Transform/GENERIC_API pipeline. Trigger on "should this be an API step or GENERIC_API", "which M3 call pattern should I use here", "why do all the real customer workflows use Transform+REST instead of the native step", or any moment about to design a new M3-touching step from scratch.
---

# Native `API` step vs. `GENERIC_API` pipeline — the decision rule

Two confirmed, real ways to call M3 from a Vince Live workflow exist side by side:
- the native `API` step (see `vince-m3-native-api-step`)
- the `GENERIC_API` Transform → REST → Transform pipeline (see `vince-generic-api-step`)

Every real customer workflow reviewed so far (Homewerks, Europris, 10009-Voice, and
`operations-workflow.txt` itself) uses the pipeline. That's not an accident — it's because the pipeline
can do things the native step structurally cannot. Don't default to the pipeline out of habit, though;
ask the questions below and pick deliberately.

## Ask these questions, in order

**1. Does this need to filter rows on a value an earlier M3 call produced in this same run?**
If yes → pipeline, full stop. `GENERIC_FILTER` (the native path's only filtering mechanism) only ever
sees raw trigger rows, runs exactly once, and runs *before* the M3 step — it structurally cannot see
anything an M3 call returned. There is no native-path way around this.

**2. Does this need a numeric or date comparison gate (`>`, `<`, date-after, date-before)?**
If yes → pipeline. The native path's field validation is existence/type/mandatory-shape checking, not
comparison logic. Comparison gating belongs in a JSONata Transform or a `GENERIC_FILTER` `operator`
(see `vince-generic-filter-step`), both of which are pipeline-side.

**3. Does this need per-call filtering that varies per row (e.g. skip this specific API call for this
specific item)?**
If yes → pipeline, using `skipIfExpression` on the REST step (see `vince-generic-api-step`) —
note that field's *behavior* is customer-claimed, not independently verified executing, so test it if
this is load-bearing.

**4. None of the above — is this a straightforward "read/write these transactions" case?**
Then the native path wins on real advantages: automatic pagination, stricter save-time field
validation, and full field-metadata lookup support (see `vince-field-metadata-lookup`) so mismatched
field names get caught before the workflow ever runs. If the task is simple, prefer native — it's less
JSON to hand-write and the platform validates more of it for you.

## Why the pipeline dominates in practice anyway

Real integrations are rarely "just read these transactions" — they almost always need to join M3 data
with something else, gate on a business condition, or shape the output for a report/email. That's why
question 1 or 2 usually fires before question 4 gets a chance to matter. Treat "the pipeline is more
common" as an observed consequence of real requirements, not a rule to follow blindly when a genuinely
simple case comes up.

## What not to do

Don't pick the pipeline reflexively because it's what you've seen most, and don't pick native
reflexively because it "looks simpler" — both defaults skip the actual structural question, which is
whether filtering/gating needs to happen *after* M3 data is already in hand.
