---
name: vxl-classic
description: Use for VXL Classic - the older Vince client/server product for moving data between Excel and Infor M3, comprising the VXL Server (VPM, hosted at vpm2.vincesoftware.org) and the VXL Desktop Client. Covers functions and function versioning, environments, roles and authorities, labels, API metadata upload, trial signup and client installation, SSO and MFA support, and the Customer Migration Process moving 50+ customers from Classic to Vince Live. Use when someone mentions VXL Classic, VPM, the desktop client, or a Classic function or its configuration XML.
---

# VXL Classic

The client/server predecessor for Excel↔M3 data movement — still actively
released, and simultaneously the subject of an organised migration to Vince
Live.

## Provenance

Compiled from Vince's Notion workspace on **2026-09-22**. Notion is the source
of truth. Note that the whole product-documentation set is a Confluence→Notion
migration and shows it — the "Register for trial version" page still carries a
dead Confluence `createpage.action` link.

**The documentation never expands the acronym "VXL".** Do not guess at it.

## What it is

A client/server tool for moving data between Microsoft Excel and **Infor M3**
via M3 APIs. Two halves:

- **VXL Server**, also called **VPM**, hosted by Vince at
  `vpm2.vincesoftware.org` — where functions are authored and administered.
- **VXL Desktop Client**, a Windows ClickOnce/standalone app.

The split has an explicit stated reason:

> *"Since the VPM server cannot access the customers M3 environment, functions
> must be run in the Desktop Client."*

**Users** are M3 business/super users running export and import functions.
**Company Admins** administer users, authorities, environments and functions,
and may be internal or external users.

## Vocabulary

**Function** — the unit of work, export or import; versioned, deployed or
undeployed, documented, organised by **Labels**. **Environment** — one per M3
environment, or several per M3 environment for grouping/authority purposes.
**Role** — grants access to functions and to environments. **Company** /
**Company Admin**. **API metadata**, uploaded per environment. **Excel
template** / design file. The **online XML editor** for editing a function.
**Archived functions** (new Sep 2026). `VinceExcelConfig.xsd` — the XSD for
function configuration XML, published on the Downloads page.

## Getting it running

- **Trial**: self-signup at `https://vpm2.vincesoftware.org`.
- **Client install**: one-click ClickOnce from
  `https://vpm2.vincesoftware.org/VXLApplication/publish2.htm`. A standalone
  installer is available **only by request** to support@vince.no (current
  published build: `VXL2_LocalSetup_26.09.320.zip`).
- **Windows only.** Uninstall via Windows Settings → Apps and features.
- **Connectivity**: on-prem, the client talks to M3 either by **direct socket**
  or **Infor ION API Gateway over HTTPS**, inside the on-prem network. Cloud:
  **only** via ION API Gateway. Client↔VXL Server is always HTTPS/TLS 1.2+.
  Data at rest and in transit is encrypted, **hosted within the EU**.
- **SSO**: supported in versions released **after August 2022**; **any
  OIDC-compliant IdP** (Microsoft 365, Google, Okta, Ping, Auth0).
  **SAML 2.0 is not supported.**
- **MFA**: via SMS codes to a registered phone. **App-based TOTP is not
  supported.**
- **User provisioning is not supported.**

## What you can do — the procedure index

**Function authoring** (21 pages;
[index](https://app.notion.com/p/f3281d9f77bc4ebb95ea9e59826d2487)):
create an export function with or without a predefined Excel template · create a
function from existing configuration files · edit a function with the online XML
editor · edit the function interface from the desktop client · create a new
function version · deploy/undeploy · organise with labels · add function
documentation · advanced API search · sorting API output and output fields ·
additional filtering · create an import function · include blank input values to
clear fields in M3 · set a default value for an input field · select input
values by prompting M3 · date-picker input · import from a matrix · use VXL with
Web Services · prices and decimals.

**Administration**
([index](https://app.notion.com/p/783f802d85c2477d93e4755db4e107c9)):
Edit Company · Environment administration (create/edit/copy/delete/organise) ·
Manage labels · Roles and authorization (create/edit/delete a role, grant access
to a function, grant access to an environment) · Upload API metadata for an
environment · User maintenance.

## Gotchas and known issues

- **SAML 2.0 SSO, app-based OTP and user provisioning are all unsupported.**
- ION API Gateway TLS version *"depends on customer's configuration"* — outside
  Vince's control.
- Some Windows versions raise a **"Security Warning"** on client launch;
  unchecking "Always ask before opening this file" suppresses it.
- **Fixed in v3.2026.9.320** (worth knowing if a customer is on an older build):
  the Refresh button used to reset the Environment dropdown to the first option
  and refresh the **wrong** environment's function list; and the execution
  screen sometimes failed to open a function after valid M3 credentials were
  entered.
- M3 environment version details are now stored in the DB but are **internal
  only — not shown in the UI or any report.**

## Lifecycle — both things are true

The documentation supports two statements at once, and you should give both:

**It is actively released.** v3.2026.9.320 shipped **20 Sep 2026**, adding three
admin features: archive/restore unused functions; an audit trail for role↔user
and role↔function connect/disconnect; and moving roles, functions and program
references between environments, then retiring the source environment. Release
Notes list ~28 net-change reports back to 2017.

**And it is being migrated away from.** A formal, Migration-Team-owned runbook —
**"Customer Migration Process — VXL Classic → VL"** (v1.0, scope **50+
customers**) — moves customers onto Vince Live:

- **Phase A (prerequisites, behind a gate):** customer agrees; M3 environment
  established; VL tenant and users created; most-used functions shortlisted.
- **Phase B:** migrate the shortlisted functions; verify against the template
  and design file; flag API mandatory-field discrepancies; execute to test I/O;
  demo. Ends at handover to On-Boarding.

**Do not tell a customer Classic is dead, and do not tell them it has a future
roadmap.** Both would overstate the documentation.

## Related tooling

Vince's internal **VinceForge** tool automates part of the Classic→Live
conversion (reading a Classic function's configuration XML and writing the
equivalent Vince Live workflow). It is Migration-Team-owned and not
customer-facing. Its capability matrix is the authority on which Classic
constructs can and cannot be carried.

## Key pages

[VXL Classic (product tree)](https://app.notion.com/p/aed0f651719e4a8ba6588ee280fe72a3) ·
[Function authoring index](https://app.notion.com/p/f3281d9f77bc4ebb95ea9e59826d2487) ·
[Administration index](https://app.notion.com/p/783f802d85c2477d93e4755db4e107c9)

## What the documentation does NOT answer

- **What "VXL" stands for.**
- **No M3 version support matrix** (unlike VSE, which states one), and no
  Windows or .NET requirement for the desktop client.
- **The function XML semantics** — filter structure, transaction chaining,
  `<Export>`/`<Import>` block structure — are not summarised anywhere; only the
  bare `.xsd` is published.
- **No documented API for VXL Server itself**, and no bulk export/import of
  functions.
- **No end-of-support date**, and nothing on what happens to Classic after the
  50+ customer migration completes.
- **Nothing on performance limits** (row counts, timeouts) or error handling
  during a function run.
