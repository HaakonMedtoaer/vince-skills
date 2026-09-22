---
name: m3-business-partner-crs
description: Explains Infor M3's Business Partner module (CRS) — customer and supplier master data, addresses, contacts, and BP categorization — and how to map a business-partner lookup or sync into a Vince Live workflow. Use whenever a consultant asks about customer master data, supplier master data, BP categories/groups, pulling customer lists for a mailing or report, syncing supplier data to an external system, or looking up a customer's terms/pricing group before building an order-related workflow, even if they just say "customer data" or "supplier data" without naming CRS.
---

# M3 Business Partner (CRS) module

General Infor M3 functional knowledge, not tenant-specific confirmed fact. Program/transaction
names below (e.g. `CRS610MI`) are **typical/commonly-documented M3 names, not verified against any
specific tenant's catalog.** Before building anything against them, cross-check the exact name
against the tenant's own `m3-api-catalog-full.json` (or the `search_m3_catalog` / `get_m3_program`
tools if working inside the VinceGenerator project) — the same discipline this project applies to
Vince Live's own JSON shapes applies here to M3 program names: a plausible-looking name is not a
confirmed one.

## What the module is for

CRS is M3's shared master-data model for anyone the business transacts with. The core idea a
consultant needs internalized: **a Business Partner (BP) is not "a customer" or "a supplier" —
it's one record that can hold either role, both, or neither**, with the actual customer/supplier
transactional data living in separate but linked records (customer master, supplier master) that
both point back to the same BP number. Getting this wrong — treating "customer" and "BP" as
synonyms — is the most common modeling mistake consultants make when scoping a workflow.

Core concepts:

- **Business Partner (BP)** — the shared identity: name, general classification, tax IDs. One BP
  number.
- **Customer master** — order/invoice-relevant data hung off a BP once it's flagged as a customer:
  price list, payment terms, delivery terms, credit limit, salesperson.
- **Supplier master** — the supplier-side equivalent: purchase terms, default warehouse/buyer,
  supplier category.
- **Addresses and contacts** — a BP can have multiple addresses (invoice address vs. delivery
  address vs. head office) and multiple contact persons, each with their own role/purpose code.
- **Categories/groups** — free-form or semi-structured classification fields used to segment BPs
  for pricing, statistics, and reporting (e.g. customer category, sales district, statistics
  group). These are usually what a "pull all X category customers" report is actually filtering on.

Typically-documented programs a consultant will hear referenced (unverified against any specific
tenant, see caveat above): `CRS610MI` (customer master), `CRS620MI` (supplier master), `CRS165MI`
or similar for BP-level data, `CRS690MI`/`CRS695MI` family for addresses. Treat these as a starting
guess for where to search the catalog, never as a name to hardcode into a workflow spec.

## What a consultant typically needs to do here

- **Read a list of customers/suppliers filtered by category** — for a mailing, a report, or a
  Custom Table sync. This is a bulk read, not a single lookup: reach for the same decision this
  project already made between M3's two bulk-read paths (`DATA_LAKE` Compass SQL vs. an
  `EXPORTMI`/`Select`-style `GENERIC_API` call) — see `vince-data-lake-step` and
  `vince-exportmi-select-step` for the confirmed shapes of each. Don't build a fan-shaped `$map` of
  N individual BP lookups when the target is "all customers in category X."
- **Look up one customer's terms/pricing group before creating an order** — a single-record,
  transactional read. This is the `API`/native-M3-path case, not the bulk path — see
  `vince-m3-native-api-step` and, for the decision between native and pipeline,
  `vince-native-vs-pipeline-decision`.
- **Sync supplier master changes to an external system** — usually a scheduled `DATA_LAKE` or
  `EXPORTMI` pull of changed records (if the program exposes a last-changed-date filter) feeding a
  Transform and then whatever downstream sink (Custom Table, external REST call via `GENERIC_API`,
  email/report). Confirm whether the specific supplier-master transaction actually supports a
  changed-since filter before assuming an incremental sync is possible — that's a catalog/field-
  metadata question (`vince-field-metadata-lookup`), not something to guess.

## Where this goes wrong

- Assuming a customer number and a BP number are the same identifier space without checking —
  they're usually the same number in practice but the *relationship* (BP → customer role) is what
  actually needs to be true for a lookup to succeed.
- Filtering on a category field name without confirming it exists and holds the values the brief
  assumes — category/group fields are exactly the kind of tenant-specific configuration that
  varies wildly between customers, unlike the module's structural concepts above.
