---
name: vince-peppol
description: Use for Vince Peppol - e-invoicing and electronic document exchange between Infor M3 and the Peppol network. Covers the four-corner model and where Vince sits in it, the three flows (M3 via MEC/SFTP, M3 via ION, and inbound to M3 or AP software), the implementation steps in order, SMP and Access Point registration, the Peppol readiness check for M3, country-specific requirements, and the documented blockers such as the 6 MB payload limit, certificate renewal and 3-month archiving. Also use for PEPPOL BIS, EHF, XRechnung, SyncInvoice BOD or Access Point questions in a Vince context.
---

# Vince Peppol

E-invoicing and electronic document exchange between Infor M3 and the Peppol
network, with Vince Live as the integration and monitoring layer.

## Provenance

Compiled from Vince's Notion workspace on **2026-09-22**, principally the
*Internal Tech Documentation → Vince Peppol* tree. Notion is the source of
truth. The implementation guide is **v1.0, Oct 2025**; Vince has been **live in
PROD since 2025-04-09**.

## What it is

Vince operates as a **Peppol service provider / certified Access Point**, with
Vince Live wrapped around it for integration and monitoring.

**Positioning in the four-corner model** — be precise about this, it comes up:

- Vince Peppol operates as **C2** (the sender's Access Point) and, for
  receiving, **C3** (the receiver's Access Point).
- **Important caveat:** in practice outbound documents are *not* sent by the
  Access Point itself but by a **custom sender application (a Sender Lambda)**.
  The actual Oxalis Access Point is used **only for inbound (C3)**.
- Vince Live injects itself between C1↔C2 outbound and C3↔C4 inbound.

**Architecture** is deliberately segregated: Vince Peppol runs in a **separate
AWS account (856181577778, eu-central-1)** from Vince Live — for SLA, archiving
and data-access legal reasons, and so Peppol can be sold standalone.

> A customer **can** use Vince Peppol without being a Vince Live tenant — but
> then loses dashboards, monitoring, document search/view, retry and error
> correction, which are all Vince Live features.

## The three flows

| Flow | Expected for |
|---|---|
| **M3 (via MEC) ⇒ SFTP ⇒ Vince Live ⇒ Vince Peppol ⇒ receiver** | M3 **on-prem** and non-M3 customers |
| **M3 (via ION) ⇒ Vince Live ⇒ Vince Peppol ⇒ receiver** | M3 **Cloud Edition** |
| **Sender ⇒ Vince Peppol ⇒ Vince Live ⇒ M3** (inbound, C3) | all inbound |

- *SFTP path*: M3 → MEC generates XML → Vince SFTP → tenant-specific S3 bucket →
  a Vince Live file-write event triggers the workflow → converts to EHF/Peppol →
  Vince Peppol API → receipt → webhook back into Vince Live.
- *ION path*: ION pushes the **SyncInvoice BOD** straight to a Vince Live
  workflow endpoint.
- *Inbound*: the AP writes the file to the Peppol S3 bucket, a File Processor
  calls a **Vince Live webhook**, and a workflow converts and pushes to ION or
  M3 API calls — or to AP software (Kofax, Basware, Coupa, Ariba are named as
  targets, "via Vince Services").

**Direction vocabulary:** AR = outbound = customer invoices = M3 Accounts
Receivable. AP = inbound = supplier invoices = M3 Accounts Payable.

## Project shape and roles

The implementation guide gives a **9+ week example plan**: weeks 1–2 environment
setup, Peppol registration, initial workflow config; 3–5 outbound (AR) mapping,
transformation and validation testing; 6–8 inbound (AP) config, supplier
testing, monitoring; week 9+ end-to-end testing, UAT, go-live readiness.

*(The FAQ page contradicts this in tone, saying a system "could be running in a
day or two" technically for an existing Vince Live customer, with most time
going to tweaks. Use the 9-week plan for a real project.)*

| Party | Owns |
|---|---|
| **Customer** | project ownership — governance, supplier/customer communication, business validation, test sign-off |
| **Integration Partner** | ERP config, BOD mapping, system-level validation |
| **Vince AS** | Peppol connectivity, transformations, workflow config, monitoring, compliance validation |

The real ANZCO project instantiates this three-way split exactly.

## Implementation steps

The steps page splits **Standard vs Non-standard** flow and tags each row with
*Where* and *Who* (**Services** vs **Product**). **Setup must be done for both
M3 TEST and M3 PROD.**

### Outbound (M3 → Peppol)

1. *If SFTP*: customer generates SSH encryption keys.
2. *If SFTP*: configure the SFTP account with the customer public key (AWS
   Network Account; **Product**).
3. *If SFTP*: configure a trigger on uploaded files (Vince Live) —
   `POST https://api.vince.live/v1/subscriptions/` with a `vrn` file pattern,
   e.g. `vrn:TENANT-<id>:files:tenant:peppol/from-erp/Faktura*.xml`. Wildcards
   are allowed anywhere; **the doc warns to be as specific as possible.**
4. *If ION/M3*: API Client creation (Vince Live).
5. *If ION/M3*: ION API Gateway configured to send to the Vince Live endpoint.
6. *If ION/M3*: ION Data Flow configured to send the **Sync Invoice BOD** to
   that endpoint.
7. **Custom Table** from template — `POST /v1/custom-tables/meta`, table
   `peppol-invoices-*`, `UPSERT`, primary keys `["id","direction"]`, ~29 columns
   (status, statusDate, statusMessage, errors, documentId/Type/Time,
   buyer/supplier/invoicee/deliveryParty fields, senderId/recipientId,
   filenamePeppolXml/filenameBodXml, transactionId).
8. **Outbound Peppol Workflow** from the "Peppol - M3 To Peppol" template
   (~15 min): Converter(xml→json) → Transform 1 (the big BOD→BIS 3.0 JSONata,
   with a `config` block of **connection IDs that must be changed**) → REST 1
   (IDM PDF fetch) → Transform 2 (embed PDF base64) → Converter 2 (json→xml) →
   Transform 3 → REST 2 (send to AP Lambda `/send`) → Transform 4 (status) →
   Transform 5 / REST 3 / Transform 6 / REST 4 (archive Peppol XML + BOD XML) →
   Transform 7 → Table Updater.
9. Dashboard from template (~10 min).
10. Peppol Status Update Workflow from template (~5 min).
11. Webhook for status updates from Peppol (Vince Live).
12. Outbound Sender document in the **Peppol AP AWS account** (Product) —
    `PEPPOL_SENDER_OUTBOUND_STATUS` parameter.

### Inbound (Peppol → M3 / → AP software)

1. Define the documents to be received and the Sender ID.
   **"Requirements for new Customer"** lists exactly what to collect: company
   name, country, **main identifier** (org number or VAT number), any additional
   identifiers (GLN), **identifier code — which MUST be from the Peppol ICD code
   list**, company contact email, company website.
2. Inbound Recipient in the Peppol AP AWS account (Product) —
   `PEPPOL_AP_INBOUND_DOCUMENT` parameter.
3. Customer and documents in the **Peppol SMP** (https://ion-smp.net).
   > **Flagged in bold caps in the source: THIS CANNOT BE DONE UNTIL THE
   > PREVIOUS STEP HAS BEEN PERFORMED.** A hard sequence dependency.
4. Custom Table from template; Inbound Peppol Workflow (~15 min, manual);
   webhook for inbound documents; dashboard from template (~10 min).
5. *If ION/M3*: ION API Gateway **Authorized App** configured to allow traffic
   from Vince Live. *If AP software*: the API Connection is **custom per
   customer**.

### AP-side configuration vocabulary (Product-owned)

SSM parameters `/{env}/peppol/apiKey` (created manually),
`/{env}/peppol/senderApiUrl` (auto), and partner configs at
`/{env}/peppol/targets/{target id}/{transaction document type}` — **whose value
is a Vince Live webhook URL**, so every document type needs a webhook in Vince
Live. Target ID = the Peppol Participant ID with `:` replaced by `_`, lowercased
for text identifiers (`0192:745707327` → `0192_745707327`). Document types:
`PEPPOL_AP_INBOUND_DOCUMENT`, `..._RECEIPT`, `PEPPOL_AP_OUTBOUND_DOCUMENT`,
`..._RECEIPT`, `PEPPOL_SENDER_OUTBOUND_STATUS`. The Oxalis identifier is
`PNO000736` for both dev and prod.

## Testing and go-live

**Nine standard test scenarios:** invoice with PO; invoice without PO; supplier
without VAT number; PDF correctly stored in IDM; automatic validation and
posting; manual reprocessing via Vince Live; authorizer derived from master
data; load testing for a division/country; GL-coded non-PO invoice.

**Exit criteria:** all scenarios pass, end-to-end Peppol→ERP validation
confirmed, customer approval for production cutover.

**Peppol Readiness Check for M3** — run this early: export sample SyncInvoice
BODs from ION for both an invoice and a credit note, compare against the
"Outbound Bod Example", confirm `@type` is exactly `"INVOICE"` or
`"CREDITNOTE"`, and confirm business-critical fields exist in the mapping.

> **Unmapped fields are silently ignored during conversion** — named as a common
> issue. Extra fields are acceptable; missing ones vanish without an error.

## Gotchas and blockers

- **SMP registration ordering** — see step 3 above. Hard dependency.
- **6 MB payload limit** on the Sender API (Lambda). Larger payloads must go via
  S3; **still an open TODO**. Peppol's own SLA requires up to 2 GB / 100 MB
  depending on message type.
- **Certificate renewal**: an AP certificate update must be applied both in the
  AP software **and in every SMP entry pointing at that AP**; an SMP cert update
  only needs the SMP. Requested via Peppol Service Desk. Recorded expiries:
  **DEV 2026-11-09, PROD 2027-02-11**.
- **Country requirements (PASR)** apply automatically by jurisdiction. The
  Country Specific Requirements table covers per-country identifier schemes,
  accreditation and 5-corner needs — France PDP, UAE, Singapore, Malaysia,
  Australia/NZ, Japan, Italy accreditation; Germany Leitweg-ID; Norway ELMA;
  NL NLCIUS mandatory.
- **Archiving retention is only 3 months** by default (both Sender and Receiver
  service descriptions say so). Longer needs **Vince Vault**. The legal
  retention analysis (pre-award 5 years, post-award 3–12 months) is an **open
  TODO**.
- **`getEndpointFromParty`** in the M3→Peppol template hard-codes a per-country
  scheme map (BE 9925, CH 9951, DE 9934, ES 9920, FR 9957, IT 0211, NL 9944,
  NO 0192, PT 9940, SE 0007) and **deliberately forces an error for NZ**
  (scheme 9999, id 'UNKNOWN'); unknown countries fall back to `"0000"` /
  `"UNKNOWN"`.
- **Invalid files are not forwarded** — APs are required to validate, and
  validation errors surface in the dashboard **before** sending.
- **Supplier-side rejections**: invoices missing mandatory fields or with
  unreadable attachments are rejected, and the rejection returns via the
  supplier's own AP.

## Vocabulary

Peppol BIS Billing 3.0 · EN 16931 · PINT · EHF (NO) · XRechnung (DE) · OIOUBL
(DK) · SG-PEPPOL · A-NZ PEPPOL · UBL / CII syntaxes · AS4 · C1–C4 four-corner
(five-corner for CTC countries) · Access Point · **SMP** (Service Metadata
Publisher) · **SML** (Service Metadata Locator) · ELMA (Norwegian central SMP) ·
OpenPeppol · Peppol Authority · PASR · **ICD code list** · Participant ID /
Peppol ID · EUSR and TSR reporting · OAGIS BOD `SyncInvoice` · ION · IDM · MEC ·
CRS945 (media control) · CRS624 (supplier/authorizer) · APS450/APS455MI ·
Kofax FIN005 · Oxalis · BR-2 (buyer reference) and schematron error IDs such as
BR-S-08, BR-CL-01, PEPPOL-EN16931-P0100 · "Vince Invoice Firewall" · Vince Vault.

## Key pages

[Vince Peppol (root)](https://app.notion.com/p/1cf8e766df53806cb851cb11765bf1b0) ·
[Peppol implementation (best-practice guide v1.0)](https://app.notion.com/p/2998e766df53803ea1f8c6ea9d16e77c) ·
[Implementation steps](https://app.notion.com/p/2948e766df5380d896d9f4f9532c604e) ·
[Overview (four corners, flows)](https://app.notion.com/p/1d08e766df538033989fff1565b3c8fb) ·
[Access Point Configuration](https://app.notion.com/p/1d08e766df5380168dbdfb810983dddf) ·
[Create Peppol Custom Table](https://app.notion.com/p/2b18e766df5380199c49cc7c4626b756) ·
[Workflow Template - M3 To Peppol](https://app.notion.com/p/2b98e766df5380d3b7ffebf95176778c) ·
[Requirements for new Customer](https://app.notion.com/p/2968e766df53806283cacbf799224bac) ·
[Tenant files ⇒ Workflow trigger](https://app.notion.com/p/21d8e766df53801dae11f7853c6860e4) ·
[Sender Service Description](https://app.notion.com/p/2858e766df538038b6b2d5c71a3c3035) ·
[Receiver Service Description](https://app.notion.com/p/2858e766df538060a70bd2f740f3e003) ·
[Country Specific Requirements](https://app.notion.com/p/28d8e766df538055bca8cbde4952a042) ·
[M3 to Peppol readiness check](https://app.notion.com/p/3138e766df5380b69e39d9fb79442894) ·
[Peppol Skala - Pipeline overview (SFTP/MEC worked example)](https://app.notion.com/p/2188e766df53808fb4f2ce98e49fb739) ·
[ANZCO Project Documentation (real project)](https://app.notion.com/p/2ea8e766df53800c80b4e709961742f9) ·
[Peppol Glossary](https://app.notion.com/p/2ea8e766df53801296c2f95f415a25be) ·
[FAQ](https://app.notion.com/p/2ae8e766df538009b437d0a59612e53c) ·
[Outstanding/questions? (open TODOs)](https://app.notion.com/p/1d08e766df53806eaef4d1410d5741da)

## What the documentation does NOT answer

- **No consolidated day-one checklist for a consultant.** The steps page splits
  work between "Services" and "Product" but never says who Product is, how to
  request their steps, or the lead time for SMP/AP registration.
- **Peppol registration itself** (registering the customer as a participant,
  ELMA/DFØ, timelines) is referenced but never described as a procedure.
- **No inbound workflow template** is documented the way the outbound one is —
  the row just says "manual creation (15 min)". The only inbound mapping example
  is buried in a customer page (ANZCO PRD - AP Inbound Peppol).
- **Dashboard templates** are referenced repeatedly but no template, field list
  or location is given.
- **No status-value catalogue.** Pages use different status strings
  (`PEPPOL_SEND_OK/FAIL`, `PEPPOL_OK/PEPPOL_FAILED`, `XML_RECEIVED`,
  `PDF_RECEIVED`, `MANUAL`, and composites from Transform 4). Nothing
  reconciles them.
- **The Testing page is a handful of raw invoice numbers**, not a procedure.
- **"Non-standard flow" is a single table row** — no guidance on what makes a
  customer non-standard or how to scope it.
- Open items the docs themselves flag: >6 MB payloads, archiving/retention
  rules, backup (max 6 h data loss), whether inbound requests may be failed at
  all, failure requirements for delivery/receive, alerting and operations not
  yet established, DeliveryNonDelivery documents.
