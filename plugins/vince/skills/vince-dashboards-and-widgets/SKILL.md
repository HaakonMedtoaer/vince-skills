---
name: vince-dashboards-and-widgets
description: Use for Vince Live dashboards and widgets - building a dashboard, choosing a widget type (Info Box, Search Box, Table, Bar/Line Chart, Display Box, Workflow, Map), wiring widgets together with Linked From and Show/Hide on Event, table widget actions that run a workflow on selected rows, editable tables, multi-select, export to Excel, or dashboard edit/read permissions. Also use for questions about the Customer 360 dashboard pattern, or when someone confuses the Vince Live dashboard with the M3 H5 Vince Live Widget.
---

# Vince Live dashboards and widgets

The presentation layer of Vince Live: dashboards hold widgets, widgets read
from Custom Tables (or a workflow), and widgets can drive each other.

## Provenance

Compiled from Vince's Notion workspace on **2026-09-22**, from the
*Vince Live & VXL Live → Dashboard and Widget* tree. Notion is the source of
truth. Note that these pages describe **Vince Live 1.0 / current** behaviour and
a separate backlog exists for a 3.0 rework — no page states which version it
documents, so verify anything version-sensitive before relying on it.

## What a dashboard is

A module of Vince Live. Dashboards hold widgets that fetch data and present it.
Multiple dashboards can serve different departments. Widgets can be colour-coded
by range, and — the point most people miss — **several widgets can read the same
table and show completely different results, because filters are applied per
widget, not per table.**

## Vocabulary

Dashboard · Widget · **Data Set** (chosen from the Table Management list) ·
Dashboard Listing Page vs Dashboard Details Page · **Edit Mode Toggle** ·
**Linked From** · **Advanced Event Handling** (Show on Event / Hide on Event) ·
Editable Table · Enable Multi-Select · **Action** · Trigger Expression · Simple
vs Advanced Filter · Field Comparison · Tag Comparison · the Appearance / Data /
Filters / Details tabs.

## Widget types

From the Dashboards Overview and the new-dashboard template list:

| Widget | What it does |
|---|---|
| **Info Box** | displays a count from data |
| **Search Box** | searches in a data set |
| **Workflow** | executes a workflow |
| **Table** | tabular data, with search/sort/export |
| **Bar Chart** | — |
| **Line Chart** | — |
| **Display Box** | displays search results |
| **Map Widget** | geographic display (documented only in the Customer 360 page) |

**Not the same thing:** the **"M3 Vince Live Widget"** is an Infor M3 H5 widget
("Vince Live Widget 1.0", from the M3 Widget Catalog) that passes M3 context as
URL parameters into a Vince Live App Builder app. If someone says "the Vince
Live widget" in an M3 H5 conversation, they mean that, not a dashboard widget.

## Building one

1. Dashboard Listing Page → **Add Dashboard** → name (mandatory) + description
   (optional).
2. On the Dashboard Details Page all widget templates are shown. Click a
   template and its configuration fields appear.
3. Typically **Name** (mandatory, unique) + **Data Set** (mandatory, searchable
   list from Table Management). **Create stays disabled until both are filled.**

**Editing an existing dashboard:** it opens with widgets *disabled*. An **Edit
Mode Toggle** appears only for Edit or Full Access users. Widget counts on the
listing page update automatically.

**Permissions:**

| Level | Can do |
|---|---|
| Admin | full access by default — no explicit permission needed |
| Edit Access | modify / create / delete widgets |
| Read Access | view only — **no Edit Mode Toggle appears** |

A user reporting "I can't edit the dashboard, there's no toggle" has Read
Access. That is the symptom.

## Table widget — the one with real depth

- Search, sort, pagination, and **Export to Excel**. With a search applied,
  **the export contains only the filtered data.**
- **Editable Table** — choose which columns are editable. **Changes are UI-only
  and do NOT update backend data.** Say this out loud to customers; it reliably
  surprises them. The checkbox cannot be unselected until all selected fields
  are removed first.
- **Enable Multi-Select** — checkboxes per row, selection persists across pages,
  Select All works at page level, plus Show Selected Items.
- **Add Action** — Name (mandatory), Description, **Select Workflows**
  (mandatory; lists every workflow in the tenant), plus **Remove on Success** /
  **Remove on Failure**. Checking both removes the rows either way.
  Only the **selected** rows are passed to the workflow — and **if a row was
  edited in the UI, the edited data is what gets passed.** That is how an
  editable table becomes useful despite not persisting anything itself.

## Configuration detail worth knowing

**Dashboard listing page.** Admins see all dashboards; regular users only those
they have access to. Search is **by Dashboard Name**. Pagination shows **6
initially** with **Load More** for the next 6. The **Add Dashboard** button is
always visible to Admins, and to regular users only if they have **Write or Full
Access to any dashboard**. A duplicate name is rejected with **"A dashboard with
this name already exists."** The thumbnail shows the name, widget count, and the
**top 4 widgets by creation order**.

**Info Box.** Displays the **count of records** in the data set. Appearance tab
offers a **View Details checkbox** (default on), font sizes, a **Range Option**
(e.g. 0–100 blue/info, 100–150 yellow/warning, 150–200 red/thumbs-down) and a
**Swap Icon and Data** checkbox (default off).

**Bar Chart and Line Chart** (identical configuration):

- **X-axis**: a column (mandatory). **Show = First (default) or Last**;
  **Items max 25**, default 10.
- **Y-axis** (mandatory): **Count** (default) or **Column**, which adds a Field
  selector **restricted to numeric columns** and aggregation **Sum (default),
  Min, Max, Avg**.
- Aggregating a text field produces exactly: `Could not load data` /
  `Error: You are trying to aggregate a text field in the widget. This is not
  possible.`
- **Update Interval defaults to 60 seconds**, customisable. Background colour
  defaults to **Grey**. Field Comparison operators: `<  ≤  =  >  ≥`.

**Search Box and Display Box.** Both have a **Details** tab listing columns with
a Modify popup (checkboxes + Select All) and drag-and-drop ordering — and in
both, **the Primary Key column's checkbox is disabled**, so it always stays
visible. A fresh Display Box reads *"No records to display. Go ahead and search
for records in \"data set name\"."*

> **The Workflow widget has no real documentation** — its page body is the two
> words **"In Progress"**. Do not describe its configuration.

## Wiring widgets together

A widget's **Linked From** dropdown offers: Search widget (select), Table widget
(click), Table widget (selection), Bar chart (select).

The **Customer 360** page works a full example end to end and is the best
reference for a real dashboard:

- Search Box → Display Box, using **Show on Event / Hide on Event**
- A **Map Widget** fed by a JSONata `$linked` Transform
- An **Info Box whose data source is a Workflow** — Transform → REST API
  against `/v1/custom-tables/aggregated/...` → Transform, with outputs mapped to
  the widget's value/description fields and colour ranges set in the Appearance
  tab
- A two-table pattern (Sales Order header → lines) where the action passes
  `{ "header": {}, "body": $selected }`

## Training

"Vince Dashboards" Basic and Advanced exist as training packages (3 sessions
each on the price list). The **Vince Dashboard – Basic** package page is the only
fully drafted package in the whole training catalogue: its goal is to enable the
customer to build and maintain **their own dashboards and connected workflows**,
covering Workflows, Transform, REST API, Tables and Triggers, then building a
pre-determined dashboard from common M3 API data. Its QnA session is explicitly
**basic questions only** — not depth on the customer's own solution. Whole
process within 5 weeks of ordering. Package status is **"Needs material"**.
See the `vince-services-and-training` skill for the commercial shape.

## Key pages

- [Dashboard and Widget (index)](https://app.notion.com/p/55486263c9b7409490b395bbe791b216)
- [Dashboards Overview](https://app.notion.com/p/14b8e766df5380678830f6fae871f920)
- [Dashboard Details page](https://app.notion.com/p/f8d5bf5c1d70496ba8a20c5c100e7730)
- [Table widget](https://app.notion.com/p/1688e766df5380378221f46657bf2eff)
- [Customer 360 Dashboard Documentation](https://app.notion.com/p/1c18e766df5380f3ae4bfbb55624a245) — the worked example
- [Vince Dashboard – Basic (training)](https://app.notion.com/p/28e8e766df53806bb62dc80622b3b22b)
- [M3 Vince Live Widget How-to](https://app.notion.com/p/2df8e766df538023b99fecc58a6a2d2b) — the H5 widget, a different thing

## What the documentation does NOT answer

- **The widget inventory is inconsistent.** Dashboards Overview omits the
  Workflow widget; the Map Widget appears only in the Customer 360 page and has
  no doc page of its own. Do not present the list above as complete.
- **Advanced / Tag Comparison filtering** is deferred to "documentation in Vince
  Live" that does not exist in Notion.
- **How dashboard permissions are granted** — Read/Edit/Full Access are
  described but nothing says where they are assigned or to whom.
- **Which product version these guides describe.** A backlog item set
  ("Dashboard & Widgets Main features for Vince Live 3.0" — tabs, new grid
  component, main filter, create records from widget, dashboard in iframe)
  shows the module is changing.
- **Nothing on dashboard sharing or embedding, refresh cadence, or data volume
  limits.**
