---
name: vince-app-builder-handoff
description: Use when handing a confirmed workflow design off to Custom Tables / App Builder screens, or deciding whether an app needs tables/screens at all. Trigger on "does this app need a Custom Table", "what does the app-folder contract look like", "is workflowType EXPRESS still right", or any question referencing vince-app-generator-handoff.md.
---

# App Builder / Custom Tables handoff

## Tables and screens are optional, not required

An app need not have Custom Tables or App Builder screens at all. A workflow can be a pure
pull → transform → email report with no persisted state and no UI (see `vince-email-step`) — this
project's `layout()`/`sheets()` drop empty columns/sections correctly for that case, so a
tables-and-screens-free design is a first-class output, not a degraded one. Don't assume a table or
screen is missing from a design just because the brief didn't explicitly rule them out — check whether
the workflow actually needs persisted state or a UI surface first.

## `TABLE_UPDATER`'s `command` vs. a table's `updateType` — don't conflate them

`TABLE_UPDATER`'s step-level config field is `command`, and only `"UPDATE"` has been observed for it —
the full enum is unknown (see `vince-table-updater-step`). `updateType` is a **separate field that
lives on the table's own meta**, not on the step. These are two different fields with similar-sounding
names; treat a mention of one as unrelated to the other unless you've confirmed otherwise.

## The handoff document itself is partially superseded

`vince-app-generator-handoff.md` is the original architecture brief, and it's still authoritative for
Custom Tables' general shape and the app-folder contract. But two specific claims in it are
**contradicted by real workflow evidence**:

- its step-type list (real workflows have ten confirmed step types with specific `target` values per
  type — see the individual `vince-*-step` skills in this pool)
- its `workflowType: "EXPRESS"` claim

Where this document disagrees with a step-type skill in this pool, defer to the step-type skill —
it's grounded in real captured workflow JSON, and the handoff doc predates that evidence. Use the
handoff doc for the parts it wasn't contradicted on (Custom Tables shape, app-folder contract in
general), not as a blanket authority.

## Practical checklist before handing off

1. Does the workflow write anything that needs to persist between runs, or feed a UI? If not, no
   table/screen is needed — say so explicitly rather than adding one by default.
2. If a table is needed, is it a final sink or a queue (see the queue pattern in
   `vince-table-updater-step`)? That changes whether a read-and-delete cycle needs to exist alongside
   the write.
3. Cross-check any step-type or field claim in the handoff doc against the step-type skills in this
   pool before trusting it — the doc is historical, not current ground truth, for those two areas.
