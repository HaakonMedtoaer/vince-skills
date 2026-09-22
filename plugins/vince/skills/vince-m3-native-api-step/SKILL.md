---
name: vince-m3-native-api-step
description: Confirmed JSON shape for the native M3 API workflow step (type API, target workflow-api) — the direct-to-M3-transaction path with field-level input/output metadata, as an alternative to the GENERIC_API Transform/REST pipeline. Use when drafting or reviewing a step that calls an M3 transaction directly, when deciding whether a workflow should use native API vs. the pipeline, or when asked "how do I add/list/update an M3 record from a Live workflow" without a Transform in front of it.
---

## What this is, and how confident to be

The native `API` step (`type: "API"`, `target: "workflow-api"`) calls an M3 transaction directly, with field-level metadata attached to the step itself, instead of going through a Transform → `GENERIC_API` REST call → Transform sandwich (see `vince-generic-api-step`).

**Confidence note**: this shape comes from the peer `vince-live-workflow` skill (VinceForge's own portable reference material, built the same way this project insists on — real UI captures and real API calls) rather than from a direct tenant capture inside this project. Treat it as confirmed, but at one remove from the workflow files this project has read directly.

## Confirmed shape

`definition.stepConfig[stepId]` for an API step is a **map keyed by transaction instance id** — meaning one single step can hold many M3 transactions, each with its own config, rather than needing one step per transaction.

Each transaction entry carries:

- **`input`** — array of field objects: `name`, `descr`, `type`, `length`, `mandatory`, `source`, `value`
- **`output`** — same field object shape, for what comes back
- **`basedOnTransaction`** — chains this transaction's execution to a prior one in the same step
- **`runOnFailure`** — controls whether this transaction still runs if a prior one in the chain failed
- **`type`** — the transaction verb

### `type` is not settled — do not assume `"ADD"` always applies

The peer skill documents `type: "ADD"` as the confirmed-safe default. But a genuine tenant capture in this project (`workflow with all the steps.txt`) shows a real API step using `type: "LIST"` for what is actually a read operation — independently corroborating that `type` doesn't simply mean "the verb is always ADD." Treat `type` as a real, required field whose full enum and semantics are still open — check what a real capture uses for the verb you need (read vs. write vs. list) rather than defaulting to `ADD` on faith.

## Native vs. pipeline — don't re-derive it here

Whether to reach for this step at all instead of the `GENERIC_API` pipeline is a separate, confirmed decision rule — see `vince-native-vs-pipeline-decision`. Short version: native wins on field-metadata validation, automatic pagination, and stricter save-time validation; it loses the moment you need per-call filtering, `>`/`<` gating, or filtering on a value an earlier M3 call produced.

## What commonly follows this step

`vince-generic-filter-step` can run before the whole M3 step (once, on raw trigger data only — it cannot sit between two native API calls). `vince-excel-step` can only be attached directly after a native `API` step, never after a Transform/REST/Code chain — that sequencing rule is part of what makes native vs. pipeline a real architectural choice, not just a style preference.
