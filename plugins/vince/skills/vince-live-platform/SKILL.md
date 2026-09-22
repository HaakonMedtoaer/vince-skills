---
name: vince-live-platform
description: Use for how Vince Live and the VXL Live Excel add-in actually work as a product - what a workflow is, which component or step to reach for (Trigger, M3 API, Excel, M3 Filter, Generic Filter, File loader, Code, Converter, Table, Transform, Email, REST API), how triggers and scheduling behave, Based On / Expand / Group by on M3 API, CONO handling, and the documented gotchas such as sheet verification, column mapping breakage and beta REST API v2. Use this for functional and how-does-it-work questions; use vince-live-workflow instead when the task is authoring or verifying the raw workflow JSON.
---

# Vince Live & VXL Live — the platform, functionally

What the product is, what each component does, and the behaviours that bite.

## Which skill to use

- **This skill** — "how does Vince Live work", "which filter should I use",
  "can a transaction depend on another one", "why did my sheet break".
- **`vince-live-workflow`** — authoring or verifying the raw `POST /workflows`
  JSON, step schemas, JSONata, ION API bulk execute. That skill carries facts
  confirmed against the real API; this one carries the product documentation.
- **The `vince-*-step` family** — the confirmed JSON shape of one specific step,
  captured from real customer workflows: `vince-trigger-step`,
  `vince-m3-native-api-step`, `vince-generic-api-step`, `vince-transform-step`,
  `vince-generic-filter-step`, `vince-excel-step`, `vince-email-step`,
  `vince-sms-step`, `vince-converter-step`, `vince-data-lake-step`,
  `vince-table-updater-step`, `vince-exportmi-select-step`, plus
  `vince-native-vs-pipeline-decision` for choosing between the native and
  pipeline paths and `vince-field-metadata-lookup` for real M3 field names.
  **Reach for those whenever the question is "what JSON does this step need",
  and for this skill when it is "what does this step do and how does it
  behave".**
- **`vince-live-administration`** — tenants, connections, users, roles, SSO.
- **`vince-dashboards-and-widgets`** — the dashboard/widget layer.

## Provenance

Compiled from the Notion tree *Vince Product Documentation → Vince Live & VXL
Live (Excel Add-in)* on **2026-09-22**. Several component pages are old
(Generic Filter 2023-11, M3 API 2024-03, Group by 2023-11) and three are
**entirely empty**. Notion is the source of truth; verify anything
version-sensitive.

## What it is

**Vince Live** — a "module-based workflow engine", delivered as a modern,
serverless SaaS application. Around the engine sit user management,
notifications, data storage (Custom Tables) and connections to other systems.
The two named main components are **Excel integration** and **Dashboard and
Widget**; more modules are planned (Kanban, Item search). Vince Live can also
serve as a backend for customer applications.

**VXL Live** — the Excel add-in. The crispest official statement:

> Vince Live — the Software as a Service that processes workflows.
> VXL Live — the Excel Add-in that uses an Excel spreadsheet as input and output
> data to Vince Live workflows.

The add-in is "largely a special, embedded, version of Vince Live, with all the
same security controls as the standard browser based version", and it
communicates **only** with Vince Live over encrypted traffic — never directly
with other services.

**VXL Classic is a separate product** with its own tree and its own desktop
client. The Vince Live documentation never explains the relationship; do not
assert one beyond "a migration process exists". See the `vxl-classic` skill.

## Core vocabulary

**Workflow** — the unit of automation. Basic configuration: name, description,
status, workflow version, labels, log level, workflow type, roles. Built on a
**Workflow Designer canvas**: drag components on, then drag one component's
output onto another's input to create **links**. *"The way you drag and drop the
components determines the order of execution."*

**Trigger** — mandatory starting point. **Connection** — saved named
credentials, referenced by Connection ID (`CONNECTION-12345`). **Tenant** — the
customer container. **Environment**, **Variable**, **Groups** (bundle workflows;
at least one workflow, unique name), **Labels** (filterable on the listing
page). **M3 User ID** — tag a Vince Live user with one and M3 transactions run
as that M3 user with their permissions; otherwise the ION API Gateway service
account is the fallback. **CONO / DIVI** — M3 Company and Division, sourced
"From Client" or "Constant".

## Component catalogue

| Component | What it does |
|---|---|
| **Trigger** | starts the workflow — Manual, Scheduled, or Events & Webhooks |
| **M3 API** | select and configure M3 APIs and transactions; drag fields between transactions to chain them |
| **Excel** | the spreadsheet designer — drag API input/output fields into a sheet layout |
| **M3 Filter** | filters records *returned by* the M3 API step, before the next step |
| **Generic Filter** | decides per-row whether a record is executed, based on an Excel column |
| **File loader** | loads Excel files from SharePoint/OneDrive |
| **Code** | JavaScript step, to "extend your workflow to do almost anything" |
| **Converter** | converts between formats — JSON-to-CSV and CSV-to-JSON documented |
| **Table** | creates/configures a Custom Table to store workflow output |
| **Transform** | changes, filters or searches JSON using **JSONata** |
| **Email** | sends result files to recipients after execution |
| **REST API** | calls any endpoint (endpoint, method, headers, body, optional Connection ID) |
| **SMS** | sends a text message. Notion page is **empty** — but the JSON shape is confirmed; see `vince-sms-step` |
| **Data lake** | Compass SQL straight against M3 tables, async, for reporting-scale bulk reads. Notion page **empty** — see `vince-data-lake-step` |
| **Table updater** | writes to a Custom Table, and is also used as a **queue between runs**. Notion page **empty** — see `vince-table-updater-step` |

> **Three steps the Notion documentation does not describe at all are
> nonetheless confirmed elsewhere**, from real captured workflow JSON. Do not
> tell anyone SMS, Data lake or Table updater are undocumented — go to the
> step skills above.

### M3 API sub-features

- **Based On** — run a transaction only on **Success** or **Failure** of a named
  earlier transaction. Available from the 2nd API onward.
- **Expand** — input values in one sheet, detailed results written to separate
  result sheets. (Documented in benefit language only — no configuration steps
  or field names exist.)
- **Group by** — a checkbox on a field, so rows sharing a value are grouped.
  Works on fields from Excel, client input, constants and external APIs, and can
  also be set on an Excel column.

### Excel step toggles — all default OFF

- **Control Spreadsheet** — validates that column→header mappings still match at
  execution time.
- **Enforce Sheet Verification** — requires the active Excel sheet name to match
  the workflow's configured sheet.
- **Save backup of output** — controls whether an Excel output file is produced
  when run from VXL Live.

### Choosing a filter

| | M3 Filter | Generic Filter |
|---|---|---|
| Filters | records returned by the M3 API step | raw rows from Excel |
| Scope | that step's output | **the whole workflow** — cannot be set per API |
| Operators | Equal, Not Equal, Less/Greater Than, Less/Greater Than Equal, Is Blank, Is Not Blank, Contains, Does Not Contain | Equal, Not Equal, Less/Greater Than, Less/Greater Than Equal |
| Placement | Trigger → M3API → M3 Filter → Excel | before the M3 work |

M3 Filter columns: Description, Field, Type, Source, Logical Type, Value,
Action. Source is "From client" or "Constant"; Logical Type is AND/OR. Generic
Filter data types: Number, String, Boolean; a "From Client" value entered at
design time acts as a default.

### Table component

Record Creation Type = **Upsert / Replace / Append**. Replace deletes all
records when new ones are added; Append makes every record unique via a
timestamp. Table Name mandatory and unique; **at least one primary key
mandatory**; default is Upsert if none chosen.

### Code step

`console.log` and other native functions are **disabled**. `axios` is available.
Helper modules: `authorizedAxios` (uses a Connection to attach auth
automatically), `getContext` (contexts `env` with `apiUrl`/`apiDomain` e.g.
`https://api.vince.live`; `globals.connection`; `globals.variable`; `tenant`;
`user`), and `files` (save, saveFromUrl, list, getInfo, get, delete).
**The `files` helper is flagged "CONCEPT ONLY as of 2023-01-23"** — its current
reality is unconfirmed.

### REST API v2

Opt in with `"version": 2`. Adds `forEach` (a JSONpath to an array) and a
skip-if condition. `bodyPath` references the current record root while
`endpoint` templating wraps in `body`. **Output changes from an object to an
array** of `{headers, body}`. Binary handling via `content-disposition`: inline
(default), attachment, base64 — each returning a `fileKey`.

**It is beta.** Its own documentation warns: *"Variable naming is a little
inconsistent, so use with caution and test thoroughly"*, v3 is said to unify
naming, and one example inconsistently shows `"version": 3`.

## Gotchas the documentation explicitly calls out

- **You must select APIs before you can use the spreadsheet designer.**
- **Scheduled and Event triggers cannot take manual input.** Neither has client
  input fields — the workflow must contain everything it needs, with defaults.
- **VXL Live always works with the currently active sheet.** Running from the
  wrong sheet "can have severe consequences, including data corruption from API
  responses". Hence Enforce Sheet Verification. Error:
  *"The active sheet 'sheet name' does not match the workflow sheet name..."*
- **Inserting or deleting a column silently breaks mapping.** Control
  Spreadsheet catches it: *"The column 'C' is mapped with the header Facility in
  workflow but found Warehouse in sheet."* **A column added at the end does not
  error.**
- **Generic Filter is workflow-wide** — "cannot be set on specific APIs".
- **M3 Filter** needs at least one mapped field, and a Logical Type once there
  are more than two filters. Date fields disable Contains / Does Not Contain.
- **Email CC/BCC recipients do not receive the mail** — stated known issue.
  Attachment type must be **File** for an Excel step and **Data** for M3 Filter,
  REST API, Transform or Code; choosing File for a non-Excel step makes the
  workflow **fail at execution**.
- **File Loader: only Excel files**, and for automated configuration **only
  SharePoint sites** — personal OneDrive and shared folders are unsupported.
  Needs Microsoft Entra admin consent. With several files in the folder, the
  most recent is chosen.
- **Table component**: "Add columns to the left/right" appear in the menu but are
  **under development and do not function in production**. A table only appears
  in Table Management *after* the workflow has been executed.
- **CONO precedence**: configured in both the workflow and the Connection? **The
  Connection's CONO wins.**
- **Code step**: public files cannot be un-published (only deleted); existing
  files cannot be made public (re-save instead); files cannot be updated, only
  overwritten with `force: true`; `/public` contents cannot be listed.
- Invalid workflows show an **orange error icon instead of a Run button**.

## Key pages

Component docs: [Workflow Components index](https://app.notion.com/p/0f16d11915ad4edc87d3b880f3dce059) ·
[How to Build](https://app.notion.com/p/d5eeb794baa842b398159521b593aea9) ·
[Trigger](https://app.notion.com/p/205cfbc6540c4809a9d216f1c62e15ad) ·
[M3 API](https://app.notion.com/p/8d8a21ba8e314e8aa308631e707a0cb8) ·
[Based On](https://app.notion.com/p/11f3926e536c4f2eab65fdadf5b91bb8) ·
[Expand](https://app.notion.com/p/26e37c47ae8c482f8de4416b8bc611c1) ·
[Group by](https://app.notion.com/p/171e0ee79862467189d18e3879e15681) ·
[Excel](https://app.notion.com/p/6731f4610a9647d2ba358d207360d35d) ·
[Control Spreadsheet](https://app.notion.com/p/cca6597884e64ab79941a2e06f8e3304) ·
[Enforce Sheet Verification](https://app.notion.com/p/6587ca464c714158abee9461018f549d) ·
[M3 Filter](https://app.notion.com/p/2e2fd20b73924875a736660918fbdf43) ·
[Generic Filter](https://app.notion.com/p/829213eee1024fddb2c15073f5542d42) ·
[File loader](https://app.notion.com/p/b3c2c293154f43b1a681c2a510d34832) ·
[Code](https://app.notion.com/p/f1c8165f3b9c4622aeb039e2f09a8c69) ·
[Converter](https://app.notion.com/p/335b4e2cf1594892a5cf92646e4ee54c) ·
[Table](https://app.notion.com/p/8507a1efe2b249f59ad8a9f3499d6dd1) ·
[Transform](https://app.notion.com/p/a1494c4573984d15a5fae207fd36f3e0) ·
[Email](https://app.notion.com/p/7008b3b8b91247499c52f3ec1e700b94) ·
[Rest API](https://app.notion.com/p/05ea5977f11d4b8f88148bf68b101917) ·
[Rest API v2](https://app.notion.com/p/10e8e766df5380c68969d8ce04614f97)

How-tos: [How-to index](https://app.notion.com/p/4f6f890b8bc04b948d61c5959a8fec83) ·
[Multiple M3 API calls in 1 REST API call](https://app.notion.com/p/9d77c239e1894973bf2abf87c9adcf5d) ·
[Export data from REST API to VXL Live](https://app.notion.com/p/9c9c97a508994b64a8f8e021b1d68645) ·
[Split processing into batches](https://app.notion.com/p/a644054aca0649bda8f57589b13c8149) ·
[Connecting ION to Vince Live](https://app.notion.com/p/13d8e766df5380978772e0f930118c75) ·
[Outbound Document Flow (BOD to Workflow)](https://app.notion.com/p/13d8e766df5380ffa74ed31748833af1) ·
[Download workflow data file](https://app.notion.com/p/fdb2e91a5a524ba283479238728cc5ce) ·
[Manage tags](https://app.notion.com/p/66d55c7159774f719b56fc9cc196017a) ·
[Data Lake Usage Overview](https://app.notion.com/p/3288e766df5380bb8fb9c4ffc6745602)

Other: [About](https://app.notion.com/p/0cdf451a9470421589cb9419505bf790) ·
[Workflow Listing Page](https://app.notion.com/p/1f18e766df5380ec85ffceced3d1f7bc) ·
[CONO Implementation](https://app.notion.com/p/1598e766df5380f6a28ee1ec5d12f91b) ·
[FAQ](https://app.notion.com/p/3450ffc2117a495893bb7de0aa4fbd60)

## What the documentation does NOT answer

- **SMS, Data lake and Table updater have no *Notion* documentation** — those
  pages are titled "(To be described)" and are empty. Their JSON shapes *are*
  confirmed from real captures: see `vince-sms-step`, `vince-data-lake-step`
  and `vince-table-updater-step`. What is genuinely missing is the functional
  description — what the product does with them, and their UI.
- **File management** is referenced from the Code page as a section that is
  "TO BE WRITTEN", and its link points at google.com.
- **Workflow types, log levels and workflow versions** are named in the basic
  configuration list but never defined or enumerated.
- **Event trigger conditions** are described only by example (new record, field
  updated, webhook) — no enumeration of supported event types or condition
  syntax.
- **Expand** has no configuration documentation.
- The Workflow Listing Page ends by referring to "known defects related to
  permission handling" **without listing them**.
