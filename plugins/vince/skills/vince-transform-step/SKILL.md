---
name: vince-transform-step
description: Confirmed JSONata patterns and context-reading rules for the Vince Live TRANSFORMER_MORPH workflow step (target workflow-transform), the step that reshapes data between a trigger or API call and whatever consumes it. Use when writing or reviewing a Transform's JSONata, when a step can't find data from an earlier step, or when asked to write JSONata for a Vince Live workflow.
---

# `TRANSFORMER_MORPH` (Transform) step

`type: "TRANSFORMER_MORPH"`, `target: "workflow-transform"`. Config: `template` (a JSONata string) and
`trimAll`. Captured from real tenant workflows.

`definitionAsl` is compiled by the platform from the workflow — never write it by hand.

## Reading context

- An earlier step: `$context.data.all.<stepId>.body.results.records` — e.g. a REST step is read as
  `$context.data.all.rest_api_1.body.results.records`, exactly as the captured workflow does.
- Trigger input: `$context.data.trigger.body.<field>`.
- The shorthand `body`/`header` means the previous step's output; after a `GENERIC_API` step that
  shorthand nests one level deeper (`body.body`). This applies to the shorthand only.

## JSONata seen in live workflows

- `$map`, including a fan-out emitting `{program, maxReturnedRecords, transactions: [...]}` for one
  batched `GENERIC_API` call
- variable binding: `$data := $context.data.trigger.body[0]`
- `$map(records, function($v, $i){ … })` with an index
- ternaries `cond ? a : b`, and `cond ? {…} : undefined`
- multi-statement blocks: `;`-separated bindings inside `( … )`
- comments `/* … */`
- `$distinct`, `$merge`, `$keys`, `$lookup`, `$split`
- indexed iteration to skip a header row: `records#$i[$i > 0]`
- user-defined functions bound with `:=` (e.g. a reusable `$formatDate`)

Vince Live's runtime hasn't been checked against the full JSONata spec, so treat features outside this
list as unconfirmed until you've seen them work.

## A shared idiom

Several unrelated customers unpack `EXPORTMI`/`Select` output with the same `$split`/`$map`/`$merge`
snippet — reuse it (`vince-exportmi-select-step`).

## Other steps use other languages

`GENERIC_FILTER` values use double braces (`{{ header.Filter_A }}`); `EMAIL` uses full Handlebars
(`{{{body.subject}}}`, `{{#if}}`, `{{#each}}`). Don't write JSONata in them.

## Not known

- What `trimAll` does exactly.
- Which JSONata version the runtime implements.
