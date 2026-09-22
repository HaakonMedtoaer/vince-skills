---
name: vse-vince-security
description: Use for VSE (Vince Security) - the web application for creating and maintaining security in Infor M3, covering user, role and function authorisation, its SOD, Review, AD and API modules, the Audit function-usage analysis, installation prerequisites for on-prem (VSE001MI via LCM) and M3CE (ION API Gateway), and its reports. Use when someone mentions VSE, Vince Security, M3 role administration, MNS150/MNS110/SES005/CSYSTR, or a periodic access review. For interpreting an SoD violation report and role-restructuring methodology, use m3-sod-analysis instead.
---

# VSE — Vince Security

The web application for creating, configuring and maintaining security in
Infor M3.

## Read this first: two different things are called "Vince Security"

The documentation describes **VSE** as a web application installed on a
**customer Windows server** with IIS and SQL Server, connecting to one M3
instance, with its own **SOD module**.

Separately, the `m3-sod-analysis` skill describes a Vince SoD product that
**runs as a Vince Live tenant** with 13 workflows and `sod-poc-*` custom tables.

**These are not obviously the same product, and nothing found reconciles them.**
They may be two generations, or two distinct offerings. Before advising a
customer on "Vince Security" or "our SoD product", establish which one is meant.
Do not assume, and do not present one product's architecture as the other's.

- **This skill** — VSE the installed product: modules, setup, reports.
- **`m3-sod-analysis`** — interpreting an SoD violation report and the
  role-restructuring methodology.

## Provenance

Compiled from Vince's Notion workspace on **2026-09-22**. Notion is the source
of truth. Duplicate VSE trees exist (one under *Vince Product Documentation /
VSE*, one at a bare `VSE / ...` path); **cite the product-tree URLs.** None of
the pages are marked verified.

## What it is

A web application for **security in Infor M3** — user, role and function
authorisation, Segregation of Duties, periodic access review, Active Directory
integration and API security. The acronym is stated outright: *"Vince Security,
henceforth called VSE."*

Stated audience: *"advanced users or super users that have been through basic
training in VSE"* — in practice M3 security administrators, plus department
managers acting as approvers in the Review module.

## Structure and vocabulary

Three sections — **VSE Settings**, **Look-ups**, **Modules**.

Five **modules**: **VSE Standard** (Users, Roles, Functions), **SOD**,
**Review**, **AD**, **API**. The Areas page lists seven **areas**: Users, Roles,
Functions, SOD, Review, **Audit**, AD.

Two distinctions the docs draw explicitly:

- **Audit** means *analyse function usage to design better roles* — reading M3's
  **CSYSTR** table to suggest a suitable role for a user or group. The docs say
  outright: ***"is not to be confused with Audit-Trail!"***
- **Review** vocabulary: **Review Group** (often a department) · **Review Task**
  (periodic, dated, can span multiple groups) · **Approvals** (the manager
  approves or deletes each role) · **Review Finished** (locks the task) ·
  **Review Report**.

**AD** vocabulary: AD Settings · **AD Integration mapping** (AD group →
divisions + roles auto-assigned on create) · List New Users · scheduled or
manual import.

**M3 objects VSE touches by name:** **MNS150** (users) · **MNS110** (functions) ·
**MNS205** / **CRS111** (email) · **SES005** (API security) · **CSYSTR**
(function usage).

The UI is a three-card layout: users → that user's roles → that role's functions
across company-divisions.

## Installation and prerequisites

Two documented paths, both installed on a customer Windows server, with **VSE
Client and VSE Server as two separate applications on the same server**.

**Hardware (both):** Xeon processor · **RAM 32 GB or above** · **disk 200 GB**.

**Software (both):** **Windows Server 2012 or later with .NET Framework 4.8** ·
**SQL Server 2012 or later** · **IIS 8** · Chrome/Edge/Firefox.

Standard practice is **two instances** — one against M3 Prod, one against M3
Test (`VSE_PRD_DB`, `VSE_TST_DB`) — on one Windows server or two. VSE Server
connects to MS SQL with a **SQL Server–authenticated user** holding admin rights
on those databases. HTTPS only if the customer supplies a certificate. Ports are
defaults but changeable.

**On-prem:** architecture is REST API to M3. *"Applicable for M3 13.4 and
service packs"* — **the only M3 version number stated anywhere.** Requires
Vince's own M3 API **VSE001MI** plus its objects and metadata (**MRS001**),
shipped as a zip, installed as a fix via **LifeCycle Manager (LCM)**, with
metadata via **RepFix** in the API Toolkit. *"There will be no changes to your
environment other than the new API."* **Install in Test before Production.**
Needs a **Service User** with **MangoAdmin/SuperAdmin** authority and permission
to all Companies and Divisions — though the elevated rights are only needed for
Last Login Date, and only if SmartOffice is in use.

**M3CE / cloud:** connects via **ION API Gateway**. The customer must supply a
`*.ionapi` or `*.json` file plus the token authentication endpoint, Client ID,
Client Secret, M3 CE REST URL, and ION service account username and password.

**Vince consultants require:** server file/folder access, IIS access, DB access
via the VSE system user, SSMS, and **M3 admin access**.

> **Each VSE installation connects to only one M3 instance.** More M3 instances
> means more VSE installations. Plan and price accordingly.

## Key procedures

- [Pre-requisites by M3 environment](https://app.notion.com/p/017f10ccefb84cc1a66c536e5fb29eba)
- [Settings](https://app.notion.com/p/c089cf407f04422f830e8c4d30277528) — 12
  areas: Settings, M3 Connection, User M3 Connection, AD, Mail Server, Refresh
  User Data, Reset User Role Data, Report Fields, VSE Users, Edit Admin Profile,
  User Preferences, API Special Settings
- [Configure the M3 connection](https://app.notion.com/p/8686ba2b682b489c9d47b6e3401a0488) —
  REST URL, M3 user/password; choose **CRS111 or MNS205** as the email source
- [Create M3 users](https://app.notion.com/p/bd4beee08ae44529a81f4cdb19300271) —
  three ways: from scratch, copy an existing user, or enable an AD user
- [Run and interpret SOD](https://app.notion.com/p/53bbb9f2389247d99222049e2799aaac) —
  ships with a **default rule set for any M3 ERP setup**; custom rules supported
- [Run an access review](https://app.notion.com/p/bd92860e4f6c4450ad5c92c6abeb64c1) —
  create Review Groups (manual or Excel import template), add users (also
  importable), create dated Review Tasks, managers approve/delete roles, optional
  "Review Finished" lock, Review Report
- [AD module](https://app.notion.com/p/bfd390626c0143a6b9b6fe4ba5b13e0c) —
  connect AD Standard **or Azure AD**, map AD groups to divisions + roles,
  schedule or manually run user import, then Create User with user type, license
  and optional API access
- [Audit](https://app.notion.com/p/d246d140563a4e8b9948e7ce496ef658) — use
  CSYSTR usage data to propose better roles
- [Reports](https://app.notion.com/p/f72258cd72494344b58ebbde9fd457b7) — Excel or
  PDF: five User reports, two Role reports, a Function report, three SOD reports,
  the Review report, and Used Functions, with full column lists per report

## Gotchas

- **One VSE installation per M3 instance** (above).
- **Functions cannot be created in VSE** — *"VSE can only use functions that
  exist in M3"*. They are imported from **MNS110**.
- **Deleting a user in VSE does not delete them in M3.** The M3 user is moved to
  **status 90** in MNS150. Re-creating a user whose credentials exist at status
  90 prompts *"The user exists in M3 with status 90, Do you want to re-activate
  the user?"*; Yes moves them to status 20 and re-imports.
- On user creation VSE tries to write the email address to **both CRS111 and
  MNS205** — but only if "from e-address" and "subject" are configured.
- The **Review module is optional.** Its "Create and connect review groups" AD
  checkbox only works if **the Review Group has the same name as the manager**.
- **"Last Login Date"** from M3 requires the elevated service user **and** only
  applies if SmartOffice is in use.
- **API access on a user** is only applicable if you have the **API module**.
- **Report 1.4** (User License Details – Roles/Function) is large and
  **generates multiple Excel files**.

## Lifecycle status — unstated

**The documentation makes no statement.** No deprecation, no successor, no
replacement named. Signals only: Net Change Reports run v1.0.0 → v1.8.0 (index
last edited Dec 2023), while content pages were edited as recently as 2026
(Reports, Areas, Review), and Vince Security still appears in the 2026
Professional Services Price List and training catalogue.

Treat the status as **unstated** — do not present it as either current or
legacy.

## What the documentation does NOT answer

- **M3 version support beyond the single "M3 13.4 and service packs" line** on
  the on-prem page. Nothing for M3CE, nothing about newer on-prem releases.
- **The API module and Look-Ups have no real content** — API Special Settings is
  only a list item.
- **No upgrade or patching procedure, no backup/restore guidance**, and no
  rationale behind the 32 GB / 200 GB sizing.
- **Default port numbers are promised** ("The following ports are used by
  default") but **the list is missing from both prerequisite pages**.
- **No SSO/OIDC story for VSE itself** (unlike VXL Classic) — only AD/Azure AD
  for the managed M3 users.
- **No troubleshooting content**: what a failed VSE001MI install, a broken ION
  connection, or a failed AD import actually looks like.
- **No licensing model explanation**, though "License Type" is a first-class
  field in five reports.
