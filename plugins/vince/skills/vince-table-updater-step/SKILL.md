---
name: vince-table-updater-step
description: Confirmed JSON shape and real usage patterns of the Vince Live TABLE_UPDATER workflow step (target workflow-table-updater), including its use as scratch/queue storage between workflow runs, not just as a final sink. Use whenever drafting or reviewing a workflow step that writes to a Custom Table, whenever someone asks "how do I write to a Custom Table from a workflow" or "would this TABLE_UPDATER JSON work," or whenever a workflow needs to process more rows than fit in one run's runtime budget.
---

# TABLE_UPDATER step

`type: "TABLE_UPDATER"`, `target: "workflow-table-updater"`.

## Confirmed config shape

`definition.stepConfig[stepId]`:

```
{
  "customTableName": ...,
  "command": ...,
  "primaryKeys": [ ... ]
}
```

Only `command: "UPDATE"` has been directly observed — the full enum of possible `command` values is **unconfirmed**. Do not assume `"INSERT"`, `"DELETE"`, or `"UPSERT"` exist as values unless you see them in a real capture; flag any need for those as an open question.

**Don't confuse this with the Custom Table meta's `updateType` field** — that's a separate concept on the table definition itself, not this step config. They look similar but are not the same thing.

## Not just a final sink — a queue pattern

`TABLE_UPDATER` is confirmed used as scratch/queue storage between workflow runs, not only as where a workflow's final output lands. Europris's invoice-reminder chain is the confirmed example: a single workflow run can't process 1,000+ invoices inside its runtime budget, so it:

1. Writes matched rows to a Custom Table (via `TABLE_UPDATER` or an `API` step).
2. A second, frequently-scheduled workflow reads up to 100 rows at a time from Vince Live's own Custom Table search endpoint — `GET .../custom-tables/search/<table>?q=*&size=100` — via an ordinary `GENERIC_API` call (see `vince-custom-tables-search-api` for that endpoint's own confirmed paging bugs).
3. Processes each row, then deletes it via another `GENERIC_API` call to the same Custom Table API, one row at a time.

This whole queue is built entirely from `GENERIC_API` calls to Vince Live's own Custom Table endpoints, plus one `TABLE_UPDATER` (or `API`) step for the initial write — worth recognizing as a deliberate pattern when a brief says something like "there could be thousands of records, process them over time" rather than assuming a single workflow run must do it all at once.

## A resolved contradiction worth knowing about

A peer confirmed-facts source (the `anthropic-skills:vince-live-workflow` skill) records this step's JSON shape as unconfirmed — "no real capture exists." That gap was in that source's own capture history, not a real platform unknown: a genuine GET capture of a reference workflow (`workflow with all the steps.txt`) contains a real `stepConfig.table_updater_1` entry, and its envelope's `createdBy` field matches the tenant token's own JWT `sub` claim exactly — proof this is a real capture from the user's own tenant, not a hand-assembled reference doc. So `{customTableName, command, primaryKeys[]}` stands as confirmed. This is a useful worked example of weighing two disagreeing confirmed-fact sources against each other rather than picking one by default.

## When to reach for this skill

- Drafting or reviewing any step that writes to a Custom Table.
- A brief implies "there are too many rows for one run" — recognize the queue pattern above rather than assuming a batch-in-one-call design.
