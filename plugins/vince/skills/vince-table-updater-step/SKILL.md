---
name: vince-table-updater-step
description: Confirmed JSON shape and real usage of the Vince Live TABLE_UPDATER workflow step (target workflow-table-updater), including its use as a queue between workflow runs. Use when a workflow writes to a Custom Table, when asked "would this TABLE_UPDATER JSON work", or when a workflow must process more rows than one run's time budget allows.
---

# `TABLE_UPDATER` step

`type: "TABLE_UPDATER"`, `target: "workflow-table-updater"`. Shape captured from a real tenant
workflow. (VinceForge's reference lists it as unconfirmed; the capture settles it.)

## Shape

```
{ "customTableName": …, "command": …, "primaryKeys": [ … ] }
```

Only `command: "UPDATE"` has been seen. Don't assume `INSERT`, `DELETE` or `UPSERT` exist as values;
flag any need for them. `command` is a step field — different from a Custom Table meta's
`updateType`.

## As a queue between runs (Europris, live)

A run that can't process 1,000+ invoices in its time budget:

1. writes the matching rows to a Custom Table (this step, or an `API` step);
2. a second, frequently scheduled workflow reads up to 100 rows over REST
   (`GET …/custom-tables/search/<table>?q=*&size=100` — read `vince-custom-tables-search-api` first);
3. processes each row and deletes it with another REST call.

Recognise this pattern whenever a brief says "there could be thousands — work through them over time."

## From Vince's product documentation (claims, not captures)

- Tables are `UPSERT`, `REPLACE` or `APPEND`. At least one primary key is required, and primary keys
  **can't be changed** after creation — a new key means a new table.
- Looking rows up by primary key works on `UPSERT` tables only.
- A deleted table's name can't be reused for up to 6 hours.

## Not known

- The full set of `command` values.
- How the step behaves when a row with the same primary key already exists.
