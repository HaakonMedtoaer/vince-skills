---
name: vince-transform-step
description: Confirmed JSONata patterns and context-reading rules for the TRANSFORMER_MORPH workflow step (target workflow-transform) — the reshaping step between a trigger/API call and whatever consumes its output. Use whenever drafting or reviewing a Transform step's JSONata template, debugging why a step can't find data from a previous step, or asked to write JSONata for a Vince Live workflow.
---

## What this is

`type: "TRANSFORMER_MORPH"`, `target: "workflow-transform"`. Config: `template` (a JSONata string) and `trimAll`. This is the step that reshapes data between everything else — building request bodies before a `GENERIC_API`/native `API` call, and reading responses back afterward.

**`definitionAsl` is compiled by the platform — never hand-author it.** You write `template`; the platform compiles the executable form.

## Reading context: the rules that actually matter

- A prior step's output: `$context.data.all.<stepId>.body.results.records`
- Trigger input: `$context.data.trigger.body.<field>`
- **Exception**: `GENERIC_API` steps nest one level deeper (`body.body`) — see `vince-generic-api-step` for why. Don't apply that exception to non-REST steps.

## Confirmed JSONata patterns seen in real production workflows

This is the richest body of confirmed detail in the project — worth knowing the specific patterns rather than treating JSONata as a black box:

- **`$map`** for reshaping arrays, including fan-shaped batching that emits `{program, maxReturnedRecords, transactions: [...]}` for a single `GENERIC_API` call carrying N M3 transactions.
- **Variable binding**: `$data := $context.data.trigger.body[0]`
- **`$map` with an index parameter**: `$map(records, function($v, $i){ ... })`
- **Inline ternaries**: `cond ? a : b`
- **Multi-statement blocks**: `;`-separated bindings inside `(...)`
- **Comments**: `/* like this */`
- **`$distinct`, `$merge`, `$keys`, `$lookup`**
- **Indexed iteration to skip a header row**: `records#$i[$i>0]`
- **Nested nullable ternaries**: `cond ? {...} : undefined`
- **User-defined functions**: bound with `:=`, e.g. a reusable `$formatDate` closure

None of these are guesses — each has been observed in a real customer workflow. If you need a JSONata feature not on this list, treat it as unconfirmed for this platform until you've seen it used, even if it's valid JSONata in general — Vince Live's own transform runtime hasn't been checked against the full JSONata spec.

## A recurring idiom worth reusing, not reinventing

Several independent customers use the **identical** `$split`/`$map`/`$merge` snippet to unpack the `REPL`-string output of an `EXPORTMI`/`Select` bulk read — strong evidence this is a semi-standard, copy-pasted idiom at Vince rather than something to derive from scratch each time. See `vince-exportmi-select-step` for the full shape of that pattern; reach for the same idiom rather than writing a new one.

## Templating language varies by step — don't cross-apply

JSONata is Transform's language only. `GENERIC_FILTER` uses double-brace Handlebars-lite (`{{ header.Filter_A }}`), `EMAIL` uses full Handlebars including triple-brace value interpolation and block helpers (`{{{body.subject}}}`, `{{#if}}`/`{{#each}}`). Three different templating languages appear across one workflow — see `vince-generic-filter-step` and `vince-email-step` for those, rather than assuming JSONata syntax works there.
