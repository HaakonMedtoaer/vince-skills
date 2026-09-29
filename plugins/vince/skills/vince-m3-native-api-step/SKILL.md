---
name: vince-m3-native-api-step
description: Confirmed JSON shape of the native M3 API workflow step (type API, target workflow-api) — calling M3 transactions directly with per-field input/output metadata, as an alternative to the GENERIC_API pipeline. Use when drafting or reviewing a step that calls an M3 transaction directly, when asked how to add, list or update an M3 record from a Live workflow without a Transform in front, or when deciding between native and pipeline.
---

# Native `API` step

`type: "API"`, `target: "workflow-api"`. Calls M3 transactions directly, with field metadata held on
the step, instead of the Transform → `GENERIC_API` → Transform pipeline.

Evidence: shape from VinceForge's captured reference (`anthropic-skills:vince-live-workflow`), plus a
captured reference workflow from a Vince tenant containing one API step.

## Shape

`definition.stepConfig[stepId]` is a **map keyed by transaction instance id** — one step holds many
transactions. Each entry has:

- `input[]` and `output[]` — field objects: `name`, `descr`, `type`, `length`, `mandatory`, `source`,
  `value`. Get real field names from `vince-field-metadata-lookup`, never from memory.
- `basedOnTransaction` — run this transaction only after a named earlier one in the same step.
- `runOnFailure` — whether it still runs if that earlier one failed.
- `type` — see below.

## `type` is not settled

VinceForge's reference calls `"ADD"` the safe default regardless of the transaction's real verb. The
captured reference workflow uses `"LIST"` for a read. The full set of values and what they control are
unknown: check a real capture for the kind of call you need rather than defaulting to `"ADD"`.

## What can sit around it

- `GENERIC_FILTER` **before** the step — runs once on raw trigger rows; it can't sit between two API
  calls or see M3 output (`vince-generic-filter-step`).
- `EXCEL` **directly after** it — the only place `EXCEL` saves (`vince-excel-step`).

Whether to use this step at all: `vince-native-vs-pipeline-decision`.

## Not known

- The full set of values for `type`, and what each controls.
- How `basedOnTransaction` and `runOnFailure` behave beyond a simple success/failure chain.
