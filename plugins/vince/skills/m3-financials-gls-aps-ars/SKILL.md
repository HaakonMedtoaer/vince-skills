---
name: m3-financials-gls-aps-ars
description: Explains Infor M3's Financials modules (GLS general ledger, APS accounts payable, ARS accounts receivable) — chart of accounts/dimensions, and how AP/AR transactions post to GL — and how to map an AR reminder, AP reconciliation, or GL reporting task onto a Vince Live workflow. Use whenever a consultant asks about outstanding invoices, AR balances, payment reminders, AP/PO reconciliation, GL postings, or accounting dimensions, even if they just say "invoices" or "accounting" without naming GLS/APS/ARS.
---

# M3 Financials (GLS / APS / ARS) module

General Infor M3 functional knowledge, not tenant-specific confirmed fact. Program/transaction
names are **typical/commonly-documented M3 names, not verified against any specific tenant's
catalog** — cross-check against the tenant's own `m3-api-catalog-full.json` (or `search_m3_catalog`
/ `get_m3_program`) before relying on an exact name.

## What the modules are for

These three are covered together because consultant work almost always crosses the boundary
between them:

- **GLS (General Ledger)** — the chart of accounts and the dimension structure (cost center,
  project, product group, etc. — exact dimensions vary per tenant configuration) that every
  financial posting is coded against. Consultants rarely query GLS directly except to resolve what
  an account/dimension combination means for a report.
- **APS (Accounts Payable)** — supplier invoices: registration, matching against a purchase order
  (see `m3-purchasing-pps`), approval, and payment. An AP invoice posts to GL when approved/
  finalized.
- **ARS (Accounts Receivable)** — customer invoices: issued against a sales/customer order,
  tracked open until paid, and the source of "outstanding balance" and "overdue" reporting. An AR
  invoice also posts to GL.

The thing to internalize: an invoice (AP or AR) is simultaneously a financial document (with a GL
posting) and an operational one (tied to a PO or customer order) — a consultant task that sounds
purely financial ("pull outstanding AR") often actually needs the order-side reference too (which
order, which customer contact) to be useful downstream.

## What a consultant typically needs to do here

- **Pull outstanding AR balances for a customer, for a reminder workflow** — this is exactly the
  shape of a real, confirmed production pattern already built on Vince Live: Europris's invoice-
  reminder chain reads matched/overdue invoices, can't process the full volume in one workflow run,
  so writes matches to a Custom Table as a scratch queue and a second, frequently-scheduled
  workflow drains it a page at a time, deleting each row as it's handled. See
  `vince-table-updater-step` for that queue pattern's confirmed shape — don't reinvent it. The bulk
  read itself is a `DATA_LAKE` or `EXPORTMI`/`Select` question — see `vince-data-lake-step` /
  `vince-exportmi-select-step`.
- **Reconcile AP invoices against purchase orders** — needs both APS (the invoice) and PPS (the
  PO) — see `m3-purchasing-pps` for the PO side. Typically a bulk read of both, joined in a
  Transform step (JSONata), flagging mismatches.
- **GL/dimension reporting** — usually `DATA_LAKE` Compass SQL directly against the ledger tables,
  since it's reporting-scale and read-only; rarely a candidate for the transactional `API` path.

## Where this goes wrong

- Treating "outstanding AR" as a single flat number when the real ask needs invoice-level detail
  (which invoice, how overdue, which contact to remind) — check what the downstream step (email,
  Custom Table) actually needs before deciding how much detail the read has to carry.
- Building the reminder logic as one large workflow run over the full customer base instead of the
  confirmed queue pattern — the Europris case exists specifically because that doesn't scale within
  a single workflow's runtime budget.
- Assuming AP/AR and GL account/dimension field names before checking the field-metadata endpoint
  (see `vince-field-metadata-lookup`) — dimension structures especially vary per tenant
  configuration and are not safe to assume from general M3 knowledge.
