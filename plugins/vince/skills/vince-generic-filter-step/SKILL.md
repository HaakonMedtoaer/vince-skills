---
name: vince-generic-filter-step
description: Confirmed JSON shape and structural limits of the Vince Live GENERIC_FILTER workflow step (target workflow-generic-filter). Use whenever drafting or reviewing a Vince Live workflow that needs to filter trigger rows before an M3 call, whenever someone asks "how do I filter which rows go to M3" or "would this GENERIC_FILTER JSON work," or whenever a workflow seems to need per-record conditional logic — this skill also explains why GENERIC_FILTER often can't do what people expect and when to reach for the pipeline path instead.
---

# GENERIC_FILTER step

`type: "GENERIC_FILTER"`, `target: "workflow-generic-filter"`.

## Confirmed config shape

`definition.stepConfig[stepId]` holds:

```
{
  "all": [
    { "fact": ..., "operator": ..., "value": ..., "source": ..., "from": ... }
  ]
}
```

`all[]` is a list of condition objects, each with `fact`, `operator`, `value`, `source`, `from`. This is the full confirmed shape — do not invent additional fields (e.g. an `"any"` sibling for OR-logic, extra comparison operators, or a `not` flag) unless you've seen them in a real capture. If you need OR-logic and only `all[]` is confirmed, flag that as an open question rather than guessing a shape.

## Templating language: double-brace

`GENERIC_FILTER` values reference upstream data with double-brace syntax, e.g. `{{ header.Filter_A }}`. This is one of three distinct templating languages used across a Vince Live workflow:

- `GENERIC_FILTER` → double-brace `{{ header.Filter_A }}`
- `TRANSFORMER_MORPH` → JSONata (see `vince-transform-step`)
- `EMAIL` → triple-brace / Handlebars, including block helpers (see `vince-email-step`)

Don't mix these up when drafting — using JSONata syntax inside a `GENERIC_FILTER` value, or vice versa, is a real class of mistake worth flagging in review.

## The structural limitation that matters most

Per the `anthropic-skills:vince-live-workflow` skill (a peer confirmed-facts source, cross-referenced in this project's `CLAUDE.md`):

- `GENERIC_FILTER` only ever sees **raw trigger rows** — it cannot see output from an earlier M3 call.
- It **always runs once**, before the whole M3 step — there's no per-call or per-batch filtering variant.
- It structurally **cannot filter on a value an earlier M3 call produced.**

This means: if the brief needs "call M3 to look something up, then filter based on what came back," `GENERIC_FILTER` cannot do it — that's not a config mistake, it's the step's boundary. Any of these three needs (per-call filtering, `>`/`<` gating, filtering on an earlier M3 result) forces the Transform/`GENERIC_API` pipeline path instead of the native `API` step + `GENERIC_FILTER` combination. See `vince-native-vs-pipeline-decision` for the full decision rule, and `vince-m3-native-api-step` for what the native path looks like when `GENERIC_FILTER` alone is sufficient (simple trigger-row filtering only).

## When to reach for this skill

- Drafting a workflow where trigger-submitted rows need a simple keep/drop condition before an M3 call.
- Reviewing a spec that claims `GENERIC_FILTER` will filter on an M3 API's own response — flag this as structurally wrong per above, and suggest the pipeline path instead.
