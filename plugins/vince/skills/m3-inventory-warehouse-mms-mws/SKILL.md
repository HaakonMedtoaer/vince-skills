---
name: m3-inventory-warehouse-mms-mws
description: Explains Infor M3's Inventory Management (MMS — item master, warehouse balances, stock transactions) and Warehouse Management (MWS — picking, put-away, warehouse tasks) together, since consultants almost always need both. Use when a Vince Live workflow needs stock levels, low-stock flags or stock-transaction reporting, or must reconcile a warehouse task against an inventory movement.
---

# M3 Inventory (MMS) and Warehouse Management (MWS)

General M3 functional knowledge, not tenant-specific fact. Program names below were checked against
one Vince tenant's API catalog (2026-09); check the exact name in the tenant's own catalog and its
fields with `vince-field-metadata-lookup` before designing against it.

## What they're for

MMS owns *what an item is and how much exists*; MWS owns *the physical work of moving it*. Most asks
sit at the boundary: MMS holds the balance, MWS the task about to change it. Examples: item basic data
via `MMS200MI` (`GetItmBasic`), balance identities via `MMS060MI` (`LstBalID`).

## Core structure

- **Item master vs. item/warehouse.** The item master holds what doesn't vary by location
  (description, unit, item group). The item/warehouse record holds what does: reorder point, lead
  time, planning method, balances. "What's the reorder point?" needs the item/warehouse record — it
  can differ per warehouse.
- **Three balances, often confused:** *on hand* (physically there), *allocated* (on hand but committed
  to an order) and *available* (on hand minus allocated — usually what matters for promising or
  reordering). A low-stock report on on-hand instead of available flags the wrong items.
- **Stock transactions.** Every balance change is a typed, logged transaction (receipt, issue,
  transfer, adjustment, count correction). The log answers "what changed and why" better than
  diffing snapshots.
- **Warehouse tasks (MWS).** Picking and put-away are tasks; completing one creates the stock
  transaction. Task and transaction are separate records that should agree — a mismatch is a real,
  reportable condition.

## Mapping work onto Vince Live

- **Current stock for many items/warehouses** — a bulk read: `vince-data-lake-step` or
  `vince-exportmi-select-step`, not a balance transaction per item, which won't fit a run's time
  budget at real scale.
- **Low-stock flags** — bulk read of available vs. reorder point, filtered in a JSONata **Transform**
  (`GENERIC_FILTER` can't see M3 output), sent by `EMAIL` or queued with `TABLE_UPDATER`
  (`vince-table-updater-step`) for a separate, more frequent run.
- **Task vs. transaction reconciliation** — bulk read of both, joined in a Transform; the mismatches
  are the report.

## Ask before promising a field or transaction

- Does "stock level" mean on hand, available, or available in one warehouse?
- Is the reorder point maintained in M3 or in an external planning tool? That can reverse the
  workflow's direction.
- Can this tenant's MWS setup produce a task/transaction mismatch at all, or are they atomic?

## Not known

- A tenant's warehouse structure and balance rules, and whether its MWS setup can produce task/transaction mismatches.
