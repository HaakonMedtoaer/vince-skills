---
name: vince-generic-filter-step
description: Confirmed JSON shape and structural limits of the Vince Live GENERIC_FILTER workflow step (target workflow-generic-filter). Use when a workflow must keep or drop trigger rows before an M3 call, when asked "how do I filter which rows go to M3" or "would this GENERIC_FILTER JSON work", or when a design expects a filter to act on data an M3 call returned — this skill explains why it can't.
---

# `GENERIC_FILTER` step

`type: "GENERIC_FILTER"`, `target: "workflow-generic-filter"`. Shape captured from a real tenant
workflow.

## Shape

```
{ "all": [ { "fact": …, "operator": …, "value": …, "source": …, "from": … } ] }
```

That is the whole confirmed shape. Don't add an `any` sibling for OR logic, a `not` flag or extra
fields without a capture; if a design needs OR logic, flag it as open.

Values reference upstream data with **double braces**, e.g. `{{ header.Filter_A }}` — not JSONata
(Transforms) and not full Handlebars (`EMAIL`).

## What it can't do

From VinceForge's captured reference (`anthropic-skills:vince-live-workflow`):

- It sees only **raw trigger rows**, never the output of an M3 call.
- It runs **once, before the whole M3 step** — there's no per-call variant.

So "look something up in M3, then filter on the result" is outside this step entirely — use a JSONata
Transform in the pipeline path (`vince-native-vs-pipeline-decision`). A design that puts
`GENERIC_FILTER` after a bulk read to filter its results is structurally wrong.

## Operators

The `operator` values seen in captures aren't enumerated here. Vince's product documentation lists
Equal, Not Equal, Less Than, Greater Than, Less Than Equal and Greater Than Equal, with data types
Number, String and Boolean — a documentation claim, not a capture. Note that it sits uneasily with the
rule that comparison gates need the pipeline; flag it if a design depends on either.

## Not known

- The `operator` values that appear in real captures.
- Whether any OR logic exists.
