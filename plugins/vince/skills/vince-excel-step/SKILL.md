---
name: vince-excel-step
description: Confirmed JSON shape of the Vince Live EXCEL workflow step (target workflow-excel) and the rule that it must directly follow a native M3 API step. Use when a workflow should write results to or read rows from a spreadsheet, when asked "would this EXCEL step JSON work" or "why won't my Excel step save", or when a design attaches EXCEL after a Transform, REST or Code step — that order fails to save.
---

# `EXCEL` step

`target: "workflow-excel"`, `stepId` conventionally `excel_1`.

Evidence: shape from VinceForge's captured reference (`anthropic-skills:vince-live-workflow`); use in
production at Topro (MMS090 export) and Wonderland (PDS002, `Trigger → M3 → Excel`).

## Shape

`stepConfig` is a `sourceSheet.fields` object: the sheet name, header and data rows, and a `columns`
map binding spreadsheet columns to the preceding `m3_api_1` step's input/output fields.

## Sequencing rule

`excel_1` must **directly follow a native `API` step** (`vince-m3-native-api-step`). After a
Transform, `GENERIC_API` or Code chain it fails to save. So a workflow whose data comes through the
pipeline either moves that read to the native path or doesn't use `EXCEL`.

## From Vince's product documentation (claims, not captures)

- Three toggles, all off by default: *Control Spreadsheet* (checks column→header mappings still match
  at run time), *Enforce Sheet Verification* (the active sheet name must match the workflow's), and
  *Save backup of output*.
- VXL Live always works on the **currently active sheet**; running from the wrong one can corrupt data
  — hence sheet verification.
- Inserting or deleting a column silently breaks mappings; adding one at the end doesn't.

## Not known

- The JSON field names behind the three documented toggles.
- The shape when a spreadsheet is the workflow's input rather than its output.
