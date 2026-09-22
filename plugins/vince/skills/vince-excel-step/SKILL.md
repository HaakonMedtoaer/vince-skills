---
name: vince-excel-step
description: Confirmed JSON shape of the Vince Live EXCEL workflow step (target workflow-excel), including the sequencing rule that it must follow a native M3 API step. Use whenever a brief wants a workflow to write results to a spreadsheet, whenever someone asks "would this EXCEL step JSON work" or "why won't my Excel step save," and whenever a draft attaches EXCEL after a Transform/REST-API/Code chain instead of after an API step — that ordering is confirmed to fail.
---

# EXCEL step

`target: "workflow-excel"`, `stepId` conventionally `excel_1`.

## Existence and shape are both confirmed now

This step type was previously recorded as blocked ("no real example exists"), but that's now outdated on two fronts:

1. **Existence**: two real customer solutions use it — Topro's MMS090 export writes results to an Excel sheet, and Wonderland's PDS002 workflow is `Trigger → M3 → Excel`.
2. **Shape**: confirmed via the `anthropic-skills:vince-live-workflow` skill (a peer confirmed-facts source) — `stepConfig` is a `sourceSheet.fields` object: sheet name, header/data rows, and a `columns` map binding spreadsheet columns to the preceding `m3_api_1` step's input/output fields.

Both existence and shape are confirmed — don't undersell this as "still unconfirmed-in-shape"; that was true before the skill source filled the gap, it isn't true now.

## The sequencing rule that actually matters

**`excel_1` must directly follow a native M3 `API` step** (see `vince-m3-native-api-step`). Attaching it after a Transform/`GENERIC_API`-REST/Code chain instead fails to save — this isn't a stylistic preference, it's a confirmed save-time validation constraint. If a brief's data comes from the `GENERIC_API` pipeline rather than the native `API` step, either the whole read needs to go through the native path instead, or `EXCEL` isn't a fit for that draft.

This explains Wonderland's confirmed shape: `Trigger → M3 → Excel` is exactly "trigger, then a native API step, then Excel" — not a coincidence, a requirement.

## When to reach for this skill

- A brief wants tabular M3 results written to a spreadsheet as the workflow's output.
- Reviewing a draft that places `EXCEL` after anything other than a native `API` step — flag it as a save-time failure, not a style nit.
- Mapping which fields go into which spreadsheet column — that's the `columns` map inside `sourceSheet.fields`, driven by the preceding `m3_api_1` step's confirmed input/output field objects (see `vince-m3-native-api-step`).
