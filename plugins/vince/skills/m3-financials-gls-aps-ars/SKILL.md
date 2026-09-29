---
name: m3-financials-gls-aps-ars
description: Explains Infor M3's Financials modules (GLS general ledger, APS accounts payable, ARS accounts receivable) — chart of accounts and dimensions, and how AP and AR invoices post to the ledger — and how to map AR reminders, AP reconciliation or GL reporting onto a Vince Live workflow. Use when asked about outstanding invoices, AR balances, payment reminders, AP-to-PO matching, GL postings or accounting dimensions, even if the user just says "invoices" or "accounting".
---

# M3 Financials (GLS / APS / ARS)

General M3 functional knowledge, not tenant-specific fact. Check exact program names in the tenant's
own catalog and fields with `vince-field-metadata-lookup` before designing against them.

## What the modules are for

Covered together because the work nearly always crosses between them:

- **GLS (general ledger)** — the chart of accounts and dimensions (cost center, project, product group
  — these vary per tenant) that every posting is coded against. Usually queried only to explain what
  an account/dimension combination means in a report.
- **APS (accounts payable)** — supplier invoices: registration, matching against a PO
  (`m3-purchasing-pps`), approval, payment. Posts to GL when approved.
- **ARS (accounts receivable)** — customer invoices from sales orders, tracked until paid; the source
  of outstanding and overdue reporting. Posts to GL.

An invoice is both a financial document (with a posting) and an operational one (tied to a PO or
customer order). A "purely financial" ask such as outstanding AR usually needs the order-side
reference too — which order, which contact.

## Mapping work onto Vince Live

- **Overdue AR reminders** — the shape of a live Vince solution (Europris): overdue invoices are too
  many for one run, so matches are written to a Custom Table as a queue and a frequently scheduled
  second workflow works through it, deleting each row as it's handled. Reuse that pattern
  (`vince-table-updater-step`); the bulk read is `vince-data-lake-step` or
  `vince-exportmi-select-step`.
- **AP invoices vs. POs** — bulk reads of both, joined and compared in a JSONata Transform, flagging
  mismatches.
- **GL and dimension reporting** — reporting-scale and read-only, so usually `DATA_LAKE`; rarely a
  transactional API call.

## Where this goes wrong

- Treating outstanding AR as one number when the downstream step needs invoice-level detail (which
  invoice, how overdue, whom to remind).
- Running reminders over the whole customer base in one workflow run instead of the queue pattern.
- Assuming AP, AR or ledger field and dimension names — dimension structures especially differ per
  tenant.

## Not known

- A tenant's ledger dimensions and account structure.
- Exact AP/AR program names — none were checked for this skill.
