---
name: vince-live-rest-api
description: Use for calling the Vince Live REST API from outside the product - the workflows management and execution endpoints, triggering a workflow synchronously or asynchronously, the vl-wf-control / vl-wf-header / vl-wf-execution headers, execution keys, the vl-response-transformation JSONata header, custom-tables data endpoints and their primary-key rules, plus the auto-increment, tags, variables, schedules and utils (PDF generation, holidays) APIs. Use when integrating another system with Vince Live, not when designing a workflow's internal steps.
---

# Vince Live REST API

Calling Vince Live from outside: triggering workflows, reading results, and the
supporting platform APIs.

## Which skill to use

- **This skill** — calling the platform API from another system.
- **`vince-live-workflow`** — the structure of a workflow *definition* (the
  `POST /workflows` body, step schemas, JSONata). That skill's facts are
  confirmed against the real API.
- **`vince-live-administration`** — obtaining credentials: API Clients, the
  client-credentials token flow, roles.
- **`vince-custom-tables-search-api`** — **required reading** before any paged
  read of a Custom Table (see the warning below).
- **`vince-field-metadata-lookup`** — real M3 field names, types and mandatory
  flags.
- **`vince-generic-api-step`** — calling these same endpoints *from inside* a
  workflow, which is how Custom Table reads/writes and workflow chaining are
  actually done in practice.

## Provenance

Compiled from the Notion *APIs* tree on **2026-09-22**. Several pages are
internal-style specs with unresolved placeholders, and **three core sections are
explicit stubs** (see the gaps at the end). The APIs index's "detailed
documentation" bookmark did not resolve, so an external API reference may exist
that this does not cover.

Pages use the placeholder `{{env-apiDomain}}`; the production domain seen
elsewhere is `https://api.vince.live`.

## Conventions shared across endpoints

**Standard list response:**

```json
{
  "count": 5,
  "items": [],
  "lastKey": "string",
  "scrollId": "string"
}
```

`lastKey` is returned **only when there are more results**; `scrollId` **only**
on custom-tables search pagination.

**Response transformation** — pass a [JSONata](https://jsonata.org/) expression
in the **`vl-response-transformation`** header. *"JSONata is used extensively
within Vince Live."* Use it to limit or rename fields, filter records, calculate
or aggregate, and sort.

> **Two warnings.** All transformation happens **after** data is fetched, so
> *"pagination and/or result counts might not be correct if used for filtering
> in a partial or paginated response."* And **an invalid JSONata expression is
> silently ignored and the raw response is returned** — you get no error, just
> untransformed data.

**Limits:** total request header including URL **≤ 10 kb**; response size,
before or after transformation, **≤ 6 mb**.

**Supported on:** `GET /v1/custom-tables/data` and subpaths;
`GET|POST /v1/custom-tables/search/{tableName}`; and `ANY` on `/api-clients`,
`/connections`, `/dashboards`, `/environments`, `/gateways`, `/metadata`,
`/roles`, `/tenant`, `/users`, `/workflows` and subpaths.

## Workflows API

Two perspectives: **Management** (create, maintain, update, delete) and
**Execution** (run, check status, return data, list results).

> **Name vs alias:** *"All workflows has both a `workflowName` and a
> `workflowAlias`. The name is immutable, but the alias is not."* Both are set to
> the same value on creation; **renaming changes only the alias — which is what
> the UI displays.**

**Two API versions exist.** *"The old version is eventually going to be
deprecated, so for any new integrations, use the `v1` endpoints."* But note the
catch below.

**Old version — management:** `GET /workflows` · `GET /workflows/{workflowId}` ·
`PATCH /workflows` (id in body) · `POST /workflows` (create) · `PUT /workflows`
(overwrite, id in body) · `DELETE /workflows/{workflowId}`

**Old version — execution** *(the source consistently misspells the path as
`worfklows` — reproduce it exactly or the call 404s)*:
`GET /live/worfklows/stats` · `.../{workflowId}/results` ·
`.../results/{executionId}` · `.../download-url` · `.../upload-url` ·
`POST /live/worfklows/{workflowId}` (async) ·
`POST /live/worfklows/{workflowId}/sync` (waits for the result)

**v1** combines both under `/v1/workflows/`. **Implemented:** `GET /v1/workflows/`
· `/stats` · `/{workflowId}` · `/{workflowId}/download-url` ·
`/{workflowId}/upload-url` · `/{workflowId}/stats` · `/{workflowId}/results` ·
`/results/{executionId}` · `/results/{executionId}/data`

> **Explicitly "yet to be implemented" on v1:** `POST /v1/workflows/` (create),
> `PATCH`, `PUT`, `POST .../run`, `POST .../run/sync`, `DELETE`. **So despite the
> advice to prefer v1, creating and running a workflow still goes through the
> old endpoints.** Say this rather than sending someone to a v1 endpoint that
> does not exist.

**Access:** *"Only workflows you have access to are returned when listing or
searching."* Search via the `search` query param, e.g.
`?search=workflowAlias:Finance*` and `?search=startTime:[2023-03-12 to 2023-03-13]`.

### Triggering a workflow

Allowed body mime types: `application/json`, `application/json;charset=UTF-8`,
`application/xml`, `text/xml`, `text/plain` — and the `content-type` header must
match.

> **Request size limit is 6 MB.** For larger payloads use the **`triggerFile`**
> property referencing a file uploaded via
> `/v1/workflows/{workflowId}/upload-url`. **There is no mime-type limitation on
> a file passed as `triggerFile`.**

**Execution-control headers:**

| Header | Purpose |
|---|---|
| `vl-wf-control-return-data` | `true`/`false` — use with `/sync` |
| `vl-wf-control-log-level` | log level |
| `vl-wf-control-trigger-file` | reference an uploaded file |
| `vl-wf-control-notify-on-error`, `-notification-email` | error notification |
| `vl-wf-control` | all of the above as one stringified object with camelCase keys, e.g. `{ returnData: true, logLevel: 10 }` |
| `vl-wf-header-<key>` | a value passed into the workflow trigger header — **casing is not considered for keys provided this way** |
| `vl-wf-header` | the same as a stringified **or Base64-encoded** object — *"HTTP protocol does not support non-UTF8 characters in header values, so use Base64 encoding if you expect non-UTF8 characters"* |
| `vl-wf-execution-key1` … `key9` | execution keys (see below) |
| `vl-response-transformation` | JSONata, as above |

### Execution keys

A workflow can index key data when storing results — order numbers, customer
names, external transaction IDs — so results can be searched later.

- **There are a total of 10 available keys.** Each gets a name shown in the
  Vince Live UI.
- *"Every definition must be a valid JSONata expression, which also allows for
  constant values."*
- **Values are indexed as text**, so only text or an array of strings is
  intended.
- Keys are evaluated and indexed **as soon as the data is available**.

Example expressions against a `Trigger => Rest API (get) => Transform => Rest
API (send)` workflow:
`$context.data.all.rest_api_1.orderHeader.orderNumber` ·
`$context.data.trigger.header.country` ·
`$context.data.all.transform_1.numOrderLines` · `$context.env.api.domain`.

**Configuring default execution keys is currently only available in the old
version**, via `PATCH /workflows` with an `executionKeys` object plus
`workflowId` and `tenantId`. **Runtime keys passed per-trigger via
`vl-wf-execution-key1..4` override the defaults.**

## Custom Tables API

> ### Before you write any paged read against `/custom-tables/search/`
>
> **That endpoint has two confirmed production bugs that silently corrupt
> results** — `from` is off by one against Elasticsearch's 0-indexed `from`, so
> passing `from=1` skips the true first record of every query; and climbing
> `from` across separate calls drops or duplicates rows, because the default
> sort order is not stable between calls. Neither raises an error.
>
> **Read `vince-custom-tables-search-api` before designing any multi-page
> fetch.** It carries the confirmed-safe pattern (narrow the query by prefix
> until every leaf is under the 2000-row cap, and never page at all). Those
> facts were established empirically against a real tenant; the Notion
> documentation below does not mention them.

Table types: **`APPEND`**, **`UPSERT`**, **`REPLACE`**.

- **Single record by `__id`** (the internal database id) — all three types:
  `GET /v1/custom-tables/data/{tableName}/{id}`. Responses include `__id`,
  `__created`, `__modified`, `__modifiedBy`.
- **Records by primary key values** — **UPSERT only**. Single:
  `GET /v1/custom-tables/data/{tableName}?records.{pk1}={v1}&records.{pk2}={v2}`.
  Multiple: the indexed form `records[{i}].{pk}={v}`.
  *(The stated URL-length limit reads "8192 kb", which is almost certainly a
  unit error in the source; no corrected figure is given.)*
- **Records that begin with** — **UPSERT only**:
  `GET /v1/custom-tables/data/{tableName}?beginsWith={prefix}`. Multiple keys are
  concatenated with `#`. **Output ≤ 6 mb; the prefix is case sensitive; the
  sequence of primary keys matters.**
- **Updating objects in an array**:
  `PATCH /v1/custom-tables/data/{tableName}/array-items/{pathToArray}` with
  `primaryKeyValues`, `keys`, `value`. **If no match is found in the array, it is
  appended.** Two warnings: the merged `keys` + `value` object **replaces** the
  existing object if found; and **even with a `schema` and `enforceSchema: true`,
  data validation is NOT performed on an object inserted into an array.**

**Configuration ("meta"):**

> *"All custom tables are actually schema-less, meaning they do not need any
> prior field definition, except for the fields that form the primary keys."*

- **At least one primary key is required**, and **"once defined, the primary keys
  can not be changed. If you need to add or change the primary keys, you need to
  create a new table."**
- Some UI features, **especially around Dashboards**, use the **`columnConfig`**
  section (entries of the form `{"aliasName": ..., "dataType": "STRING"}`).
- **Deleting a table has a delay of up to 6 hours** before the table is actually
  deleted, and **a new table with the same name cannot be created until then.**
  Data is inaccessible from the moment of deletion.
- Config can be updated from the UI **only if the table has no data**; otherwise
  use `PATCH /v1/custom-tables/meta`.

## M3 field metadata — confirmed, but absent from the Notion API tree

```
GET https://api.vince.live/metadata/{connectionId}?details=FIELDS%23{program}%23{transaction}
```

**The `#` separators must be percent-encoded as `%23`.** One call returns every
input/output field's `name`, `descr`, `type`, `length` and `mandatory` for that
transaction. Auth is the usual short-lived Bearer token.

This is how you avoid guessing an M3 field name. Full detail, plus the
`m3-field-catalog.sqlite` extract built from it, is in
`vince-field-metadata-lookup`.

## The supporting APIs

**Auto-increment** — number series, auto-incremented per call.
***"Operations are atomic ensuring a unique number every time."***
`GET|POST /v1/autoincrements/control` · `PUT|DELETE /v1/autoincrements/control/{name}`
· **`GET /v1/autoincrements/live/{name}`** returns `{"value": 20000001}`.
Create body: `{"autoIncrementName": "shipment-number", "initialValue": 10000000}`.

**Schedules** — *"By using this API you can create the auto schedule to run a
workflow."* `POST /v1/schedules/` with **`workflowId`**, **`expression`**
(e.g. `cron(0/3 * ? * MON-FRI *)` or `rate(5 minutes)`) and **`scheduleName`**
mandatory; optional `startDate`, `endDate`, `timezone`.
***"Have to send start and End date in UTC timings."***
Read: `GET /v1/schedules/` · `/by-workflow/{workflowId}` · `/{entityId}`.
Update `PATCH /v1/schedules/{entityId}`; delete returns **202 Accepted**.

**Variables** — stored in DynamoDB, **encrypted using the "Vince crypto
library"** and decrypted on read.
`POST /v1/api/variables/` · `GET|PUT|DELETE /v1/api/variables/{key}` ·
`GET /v1/api/variables`. Fields: **`variableName`**, **`value`**, and
**`isSecret`** (`true`/`false`) — *"used to indicate the variable to be encrypted
decrypted"*. Status codes 201/200/204/400/401/404.

**Tags** — marked **WORK IN PROGRESS**.
*"All entities in Vince Live, such as User, Tenant, Workflow, Connection, etc,
are taggable."* `{method} /v1/tags/{entityId}[/{key}]` with GET/PUT/DELETE;
entity ids like `USER-123`, `TENANT-123`, `WORKFLOW-123`. **DELETE returns no
output.**
A tag value may be `string`, `number`, `boolean`, `null`, or **an array of those
(all the same type)** — which **contradicts the UI-side Tag guide**, which says
only one value per tag is currently supported. Flag the contradiction rather
than picking one.

**Utils — PDF generation.** Engine is **[PDFme](https://pdfme.com/)**; build the
template in PDFme's template editor and download it as JSON via **DL Template**.
**V1** `POST /v1/utils/generate-pdf`, **V2** `POST /v2/utils/generate/pdf`.
Mandatory body: **`template`** and **`inputs`**.
***"For tables, note that you need to provide an array of arrays as input."***

**Utils — Holidays.** Based on the `date-holidays` NPM package.
`GET /v1/utils/holidays?country=no`. **`country` mandatory**; `state` and `year`
optional (`year` defaults to current). Items carry `date`, `start`, `end`,
`name`, `type` (e.g. `public`, `observance`), `rule`.

**Tenant — email templates.** HTML allowed, set at tenant creation or via
`PATCH /tenant`. **Currently the only template is `inviteEmail`**, sent when a
new user is created. Context: `user`, `tenant`, `code`, `urls.signIn`. Parsed
with "Vince Live Brackets" (named but never defined).

> **Gotcha:** *"`code` can contain special characters and MUST be in the template
> as `{{{ code }}}`"* — triple braces — *"so that the parser does not remove
> non-HTML compatible characters."* Get this wrong and the user cannot sign in.

## Key pages

[APIs index](https://app.notion.com/p/cf024dd2172342dc90c35449b53274e1) ·
[Common features](https://app.notion.com/p/ef4b049863d9493fa03af2956df51a0c) ·
[Workflows](https://app.notion.com/p/8bfa94e435b24d8bb53161ef3a84bde4) ·
[Custom Tables](https://app.notion.com/p/7cdf77f47c6b4e82aebbfa44fe0aef25) ·
[Auto-increment](https://app.notion.com/p/21d8e766df5380d7871cc0cbb133bbb9) ·
[Schedules](https://app.notion.com/p/1344dfb7df5949188d461acc20efb2a1) ·
[Variables](https://app.notion.com/p/8fab3c8f3ea14a1db5f3632ae9145d73) ·
[Tags](https://app.notion.com/p/e5ecd0c002074a2abd9680396748826d) ·
[Utils](https://app.notion.com/p/2f9064b652f64399a9282985ace2aa1f) ·
[Tenant](https://app.notion.com/p/05ae7c9dcc6b4e35852372d794724e13)

## What the documentation does NOT answer

- **No authentication documentation for the API itself** beyond an
  `Authorization` header appearing in one table — no token acquisition flow and
  no base domain (pages use `{{env-apiDomain}}`). Get that from the API Clients
  section of `vince-live-administration`.
- **Custom Tables has "General information: TBD", "Searching: TBD" and "Updating
  records: More to come"** — the core of that API is undocumented. The full
  `columnConfig` dataType set is never listed; only `STRING` appears.
- **Workflows leaves two sections unwritten**: *"Searching for execution key
  values — TO BE DOCUMENTED"* and *"Getting workflow results — To be written."*
- **The `/v1/workflows` search grammar** (`search=field:value`) is shown only by
  two examples; the full syntax is undocumented.
- **The `Connections` API page is entirely blank.**
- **Nothing documents failure behaviour, retries, error taxonomies or rate
  limits**, beyond the 6 MB request/response and 10 kb header ceilings.
- **"Vince Live Brackets"** is named as the template parsing syntax but never
  defined.
