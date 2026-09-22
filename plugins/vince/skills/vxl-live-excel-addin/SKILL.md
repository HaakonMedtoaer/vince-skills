---
name: vxl-live-excel-addin
description: Use for the VXL Live Excel add-in as an end-user and deployment concern - installing or deploying it via the Microsoft 365 admin center, SharePoint or side-loading, the minimum Excel versions, logging in with a tenant name or Microsoft 365 SSO, forgot-password, the All/Favorites/Recent tabs and label filters, design files, user instructions, environment colours, running a workflow, and Groups in VXL Live. Also use for the known "You don't have permission to use this add-in" failure after Excel 2507 and the Office add-in cache fix.
---

# VXL Live — the Excel add-in

The end-user and deployment side: getting the add-in installed, logged in, and
running workflows — and fixing it when Excel breaks it.

## Which skill to use

- **This skill** — installing, deploying, logging in, the add-in UI, running a
  workflow, troubleshooting the add-in.
- **`vince-live-platform`** — what workflows and components do, and how they are
  built in Vince Live.
- **`vince-live-administration`** — tenants, connections, users, roles, SSO
  configuration.

## Provenance

Compiled from Vince's Notion workspace on **2026-09-22**. All pages report
`verification: unverified`.

Note: authoring happens in **Vince Live**; **VXL Live** is the Excel add-in that
searches, reads and **executes** those workflows. The parent "VXL Live - The
Excel Add-in" page has **zero body text** — the relationship is never stated
explicitly anywhere, only inferable. Don't over-claim it.

## Installing and deploying

**Prerequisites:** a compatible Excel version (below), plus **the tenant alias
provided by your Vince AS contact** — the user enters it the first time they
start the add-in. *(One line on the install page is garbled, reading "tenant
UR*"; a later line on the same page says "tenant alias".)*

The add-in is published on the **Microsoft Marketplace**. In Excel, install via
the **Add-in Store**, searching for **"VXL Live"**.

> **Access to the Add-in Store may be restricted by company policy**, in which
> case an IT administrator must deploy it.

**Administrator deployment (the recommended route):** Microsoft 365 admin center
(`https://admin.cloud.microsoft/`) → **Settings > Integrated apps** → **Get
apps** → search **VXL Live** → **Get it now** and accept terms → **Users** tab
to assign → verify in the listing.

> **"It might take up to 24 hours before the Add-in shows up in the Excel
> client."** Set that expectation up front.

**User setup afterwards, if needed:** Excel → **Insert** → **My Add-ins** →
**Admin Managed** tab → **Refresh** → select the add-in → **Add**.

You can run **staging and production side by side within the same sheet.**

### Side-loading

From a network share, using Microsoft's own instructions. Manifest URLs:

- Staging — `https://vxl.vince.live/manifest.staging.xml`
- Production — `https://vxl.vince.live/manifest.prod.xml`

### SharePoint Online deployment — not recommended

> **"WARNING! THIS DEPLOYMENT METHOD IS NOT RECOMMENDED, DEPLOY USING THE ADMIN
> CENTRE INSTEAD."**

Its stated limitations: the manifest must be **manually updated**; the add-in is
available to **all users, with no way to restrict deployment**; it is only *made
available*, so users must add it to Excel themselves; and **"Deployments using
Sharepoint is not tested by Vince, and provided as-is."**

If someone insists: SharePoint Admin Center → **More features → Apps** →
*classic experience* → record the catalog URL of the form
`https://<TENANT>.sharepoint.com/sites/<APP CATALOGUE NAME>/` → **Apps for
Office → Upload → Files** → select the .xml. **"It might take up to 48 hours
before the Add-in is fully available."**
Users then find it under **Excel Web**: Home → **Add-ins → + More Add-ins** →
**My Organization**; or **Excel Desktop**: File → Options → **Trust Center →
Trust Center Settings → Trusted Add-in Catalogs**, pasting the catalog URL.
If the My Organization tab or the add-in is not visible: *"This will likely
resolve itself after a little while. Enabling 3. party cookies and/or disabling
tracking prevention on the site might help."*

### Minimum Excel versions

| Platform | Minimum |
|---|---|
| Office on Windows (Microsoft 365) | **Version 2102 (Build 13801.20738)** |
| Office 2021 (Volume Licensed) | **Version 2102 (Build 13801.20738)** |
| Office on Mac | **16.50** |
| Office on iPad | **16.50** |
| Office on the web | Supported |

The highest Excel API requirement set used is **ExcelApi 1.13**
(`insertWorksheetsFromBase64()`); `getRanges()` needs 1.9,
`getAbsoluteResizedRange()` 1.7, multi-row `TableRowCollection.add()` 1.4.

## Logging in

First access shows a **Welcome Screen** with a **Tenant** text box; the tenant
name comes from the **welcome email** sent when the user is registered in Vince
Live (which also carries the login username and a temporary password). The
tenant name is validated in real time.

Two login options:

1. **Username and password** — the username is validated as existing within the
   verified tenant, then the password.
2. **Microsoft 365 SSO (OIDC flow)** — redirect to Microsoft, then back.
   **If the user doesn't exist in the verified tenant, login fails.**

**Forgot Password:** link on the login screen → enter **username** → **Send
Code** → code emailed → **Code**, **New Password**, **Confirm Password**.
**Resend Code** available. Documented error strings, useful for diagnosis:

- `"Username/client ID combination not found."` — invalid username
- `"Invalid verification code provided, please try again."`
- `"Your passwords must match."`
- `"Invalid code provided, please request a code again."` — expired

**One credential set spans both products:** *"Use the updated password to log in
on VXL Live or Vince Live."*

## Using it

**Search** finds workflows and workflow groups.

**Tabs:** **All** (everything accessible to the logged-in user, with counts),
**Favorites** (toggled by the favourites icon), **Recent** — *"most recently
accessed workflows from Vince Live"*, **showing the latest nine items**.

**Filters:** the filter icon offers **Labels, Workflows, Workflow Groups**.
Workflows and Workflow Groups are **both enabled by default**. The Labels toggle
expands label checkboxes. **Filter preferences are saved to the user's profile**
and auto-applied to future searches.

**Design file:** uploaded in **Vince Live**, in the workflow's Excel component,
where it serves as a template; **downloaded in VXL Live** for use as input data.
View it via the **Excel icon next to the workflow name**.

**User instructions:** entered during workflow creation in Vince Live; read in
VXL Live via the **notes icon** next to the workflow name.

**Environment:** shown in the **bottom navigation**, **colour-coded**; hover
shows the name; the **Environment ID** appears next to the colour. The selected
environment's colour is reflected in the **background of the RUN button**. A
workflow may be associated with multiple environments.

**Running a workflow:** select it, enter required data, click the execution
button — **whose label can be set by the workflow's creator**. State machine:
**Execution → "Running" → "Successful" or "Failed"**.

**My Profile** (bottom navigation): tenant name and the user's email (both
static), **User Preferences** (Preferred Language, Date Format, Time Format),
and **Sign Out**.

**Logs:** two tabs — **Output** (each step of user actions) and **Problems**
(errors and connectivity issues), each with **Clear**. A "Report" option to
attach logs to an email is flagged as a **future enhancement, not yet built**.

## Groups in VXL Live — and why a workflow may be invisible or un-runnable

Groups are **created in Vince Live** and become available in VXL Live. A group
holds multiple workflows. Three separate gates apply, and they explain most
"I can't see it / can't run it" reports:

1. **Workflow permissions.** Admins see all groups and all workflows in them.
   Regular users see **only workflows they have permissions for** (Read, Write
   or Full Access). Groups are only visible to users with access to them.
2. **Environment access.** Worked example: WF1 → TST, WF2 → PRD. A user with TST
   access only sees TST in the group's environment list and can execute WF1.
   Attempting WF2 gives exactly:
   *"The selected environment is not linked to the current workflow. Please
   choose a valid environment."*
3. **Connection access gates the Run button.** Each environment is linked to a
   connection. **A user with environment access but no connection access gets a
   disabled Run button** — no error, just disabled. Switching environments
   updates the button state dynamically.

Environment-specific Run button colours are configured during environment
creation. Input fields behave by type: a **date** field opens a calendar; a
field with a **default value** shows it when the workflow is selected.

## Troubleshooting: add-in won't load after an Excel update

**This is the one to know.**

- **Symptom:** the add-in fails to load with **"You don't have permission to use
  this add-in"**.
- **Affected versions:** works on Excel **before 2506**; breaks on **2507 and
  later**.
- **Root cause (stated):** *"a change in how Office caches Add-ins. The issue can
  be resolved by clearing the Add-in cache, or by refreshing the Add-in."*

**Manual cache clear:** Excel → **File > Options > Trust Center > Trust Center
Settings > Trusted Add-in Catalogs** → tick **"Next time Office starts, clear
all previously-started web add-ins cache"** → OK → **close all Office
applications and wait a minute** before reopening Excel.

**Scripted clear:** a PowerShell script is published. It **must run in user
context** (ideally at logon), refuses to run while any of EXCEL, WINWORD,
POWERPNT, OUTLOOK, ONENOTE or VISIO are running, and deletes and recreates
`$env:LOCALAPPDATA\Microsoft\Office\16.0\Wef`.

**Refreshing instead:** enable the **Developer** tab (File → Options → Customize
Ribbon) → Developer → **Excel Add-ins**, then **Admin Managed tab → Refresh**.

## Key pages

[VXL Live - The Excel Add-in](https://app.notion.com/p/d52cb14bb6a0442398ea02dda9436d89) ·
[Installing VXL Live](https://app.notion.com/p/6c61e6dfe6124839870f74243ea0745a) ·
[Excel versions compatible](https://app.notion.com/p/2fd76a54135c4f4d8f7ad9ee8cbecde9) ·
[Side loading](https://app.notion.com/p/b2315ac4c81d45b9a25bdd54b28c9433) ·
[Deploy via SharePoint Online](https://app.notion.com/p/6d29e6e9e47748e5b90b27a19916b6d1) ·
[NOT ABLE TO LOAD VXL LIVE ADD-IN](https://app.notion.com/p/2468e766df5380c5b83cc36844122971) ·
[Log in](https://app.notion.com/p/a9a5c385a30a4dfe9c9890c0451ce0c1) ·
[Forgot Password](https://app.notion.com/p/1758e766df5380dfb0f5c8ebe0f4172d) ·
[The tabs](https://app.notion.com/p/0820ebe4df2846e5958f9c1492d42e92) ·
[Filters](https://app.notion.com/p/75fa8045c1764856ba717eefdd0a68ef) ·
[Design file](https://app.notion.com/p/6631466f7dbe4cbeb1ddeb7ca94dceda) ·
[Environment](https://app.notion.com/p/4e7862876dc24410b73c9b2cc94f749b) ·
[User instructions](https://app.notion.com/p/010ca55b81694e3da81c81e357c10b28) ·
[My Profile](https://app.notion.com/p/b20e6315cb134d9c833877c7de07f12c) ·
[Logs](https://app.notion.com/p/aa440d0783b24ec6b557ce4806a5c886) ·
[Workflows Execution](https://app.notion.com/p/ec34a779fd43482eabd12c31f57ad3f4) ·
[Groups in VXL Live](https://app.notion.com/p/17e8e766df5380d0be24ca32b6d88cc1)

## What the documentation does NOT answer

- **The VXL Live ↔ Vince Live relationship is never stated explicitly**, and the
  parent add-in page is empty. No page defines the two products or names Vince
  Live's URL.
- **"VXL Classic" is not mentioned anywhere in this documentation set** — no
  legacy/predecessor framing, no migration note. (Classic has its own tree; see
  the `vxl-classic` skill.)
- **Labels are never defined** — used for filtering here, but nothing says where
  they are created or how they relate to Tags.
- **Nothing describes VXL Live's own version or release cadence**, or how a user
  determines which add-in version they are running — notably unhelpful given the
  caching bug above.
- **Two profile screens disagree**: VXL Live's My Profile lists
  language/date/time; Vince Live's User Profile adds Records Per Page and names
  roles "Tenant Admin"/"TenantUser", terminology used nowhere else.
