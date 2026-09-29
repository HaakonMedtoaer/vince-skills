---
name: vince-app-builder-handoff
description: What a confirmed Vince Live workflow design needs before it is handed to Custom Tables and App Builder screens — including whether it needs tables or screens at all. Use when deciding if an app needs a Custom Table or screen, preparing a workflow to be used by App Builder, asking about the app-folder contract, or checking claims in vince-app-generator-handoff.md such as workflowType EXPRESS.
---

# Handing a design to Custom Tables and App Builder

## Tables and screens are optional

A workflow that pulls data, shapes it and emails it is a complete app with no persisted state and no
UI. Add a Custom Table only when data must persist between runs or feed a screen; add a screen only
when someone needs to see or act on it. Say explicitly when a design needs neither.

## Before a workflow becomes an App Builder resource

From Vince's App Builder documentation (a draft manual — confirm against the live docs):

- **App Builder infers a workflow's output shape from past runs.** Run the workflow **at least three
  times** with realistic inputs and check the final step's output in the logs before adding it as a
  resource — otherwise App Builder can't use it or binds to fields that don't exist.
- For M3 apps the docs recommend **one small workflow per M3 interaction** (list item groups, list
  items, update item, get price, update price) rather than one large workflow.
- Nothing updates automatically: re-prompt the app after adding a resource, changing a table's fields
  or changing a workflow's output.

## Two similar-looking fields

`TABLE_UPDATER`'s step field is `command` (only `"UPDATE"` seen; `vince-table-updater-step`). A Custom
Table meta's `updateType` is a different field on the table itself.

## `vince-app-generator-handoff.md` is partly superseded

The original architecture brief is still the reference for Custom Tables' general shape and the
app-folder contract. Its **step-type list** and its **`workflowType: "EXPRESS"`** claim are contradicted
by captured workflows — use the `vince-*-step` skills for those.

## Checklist

1. Does anything need to persist between runs or feed a UI? If not, no table or screen.
2. If there's a table, is it a final sink or a queue? A queue needs its read-and-delete cycle designed
   too.
3. Is every workflow that a screen uses ready to be run three times before it's added?

## Not known

- How rollback works after a bad publish — the manual itself says to ask an admin.
- Whether InApp roles are enforced server-side or only by the generated app.
