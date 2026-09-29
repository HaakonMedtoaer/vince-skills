---
name: m3-purchasing-pps
description: Explains Infor M3's Purchasing module (PPS) — purchase order header/line structure, requisition-to-PO flow, supplier agreements, and how a PO ties to goods receipt and supplier invoicing — and how to map purchasing work onto a Vince Live workflow. Use when scoping a workflow that reads outstanding POs, creates POs from another system or reports on receipt discrepancies, or when asked how M3 purchasing status works.
---

# M3 Purchasing (PPS)

General M3 functional knowledge, not tenant-specific fact. Program names below were checked against
one Vince tenant's API catalog (2026-09); check the exact name in the tenant's own catalog and its
fields with `vince-field-metadata-lookup` before designing against it.

## What PPS is for

The buy-side counterpart to Order Entry (`m3-order-entry-ois`): requisitions, purchase orders,
supplier agreements and goods receipt. Same header/line shape and a similar status model. Purchase
order lines are exposed through `PPS200MI` (e.g. `LstLine`).

## Core structure

- **Header / line.** Header: supplier, order date, buyer, terms. Lines: item, quantity, price,
  requested/confirmed date, warehouse. Line fields can override header defaults.
- **Requisition → PO.** Some setups raise a requisition first and convert it after approval; others
  enter POs directly. Whether a tenant uses requisitions is a configuration question to ask, not
  assume.
- **Status flow.** Roughly entered → approved/released → sent → received (fully or partly) →
  matched/invoiced → closed. A PO can stay open for a long time between partial receipts, so
  "outstanding" usually means *ordered quantity > received quantity*, not "not closed".
- **Supplier agreements.** Price can come from a negotiated agreement rather than the PO line. If a
  workflow creates POs, decide whether M3's agreement price or the source system's price wins.
- **Goods receipt** creates an inventory receipt (`m3-inventory-warehouse-mms-mws`) and drives
  three-way matching against the supplier invoice (PO vs. receipt vs. invoice). Over-receipt and price
  variance are common reports.
- **Closing.** Lines that will never be fully received usually need an explicit close; check this per
  tenant if a workflow filters on "closed".

## Mapping work onto Vince Live

- **Read outstanding POs** — a bulk read: `vince-data-lake-step` or `vince-exportmi-select-step`.
- **Create POs from another system** (a supplier portal, a replenishment run) — a transactional write:
  native `API` or pipeline, per `vince-native-vs-pipeline-decision`.
- **Report receipt discrepancies** — a bulk read of PO lines and receipts, joined and compared in a
  JSONata **Transform** (`GENERIC_FILTER` can't see M3 output), delivered by `EMAIL`.

## Ask before promising a field or transaction

- Does this tenant use requisitions?
- Should PO price come from a supplier agreement or the source system?
- Does "outstanding" mean "not fully received" or "not yet closed" for this report?

## Not known

- Whether a given tenant uses requisitions, agreement pricing or automatic line closing.
