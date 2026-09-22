---
name: vbu-vince-butler
description: Use for VBU (Vince Butler) - the Java web application for scheduled and event-driven monitoring, data collection and distribution around Infor M3. Covers duties, the four module kinds (Collectors, Converters, Listeners, Publishers), the Collect vs Consume cache pattern that turns Butler into a generic read API, dashboards and widgets, dynamic parameters, connections, and installation on Tomcat with the encryption-key warning. Use when someone mentions Vince Butler, VBU, a duty, a collector or publisher, or monitoring M3 grid/MEC/EventHub.
---

# VBU — Vince Butler

*"Vince Butler — At your service."* Scheduled and event-driven monitoring, data
collection and distribution around Infor M3 and its surrounding infrastructure.

## Provenance — and a real limitation

Compiled from Vince's Notion workspace on **2026-09-22**.

> **The Notion set is partial.** The VBU Guidelines page links "the complete
> user guide" to an **external Atlassian Confluence wiki**
> (`vinceapps.atlassian.net/wiki/spaces/PUBLIC/...`), not to Notion. Treat
> Notion as incomplete for VBU, and expect to go to Confluence for depth. Say so
> rather than answering thinly from Notion alone.

Structurally the Notion tree is broad — roughly 70–90 pages including every
collector/publisher leaf — but **shallower than it looks**: the landing page,
Modules, Duties, Dashboards, Administrator tab and Appendix are index pages with
a sentence or two. The substance concentrates in *Collect vs Consume*,
*Publishers*, *Dashboards*, the *Guidelines* subtree and the separate
installation space.

## What it is

A **Java web application** that polls or listens to sources (M3 grid, M3BE,
databases, folders, EventHub, MEC, iSeries), transforms the results, and pushes
them out (email, SMS, REST, database, disk, M3 job starts) and/or onto
**dashboards**.

Stated audience: *"advanced users or super users that have been through basic
training in Vince Butler"*. Architecture and backend are deliberately excluded
from the user guide.

## Core concepts

**Duty** — *"tasks that the Butler performs. They can either be scheduled or run
manually."*

**Modules**, of four kinds:

| Kind | Role |
|---|---|
| **Collectors** | bring data in |
| **Converters** | change format inside a duty (e.g. to XML before DiskWriter) |
| **Listeners** | receive *pushed* data from external systems (EventHub, StreamServe) |
| **Publishers** | send data out |

### Collect vs Consume — the load-bearing concept

Every duty writes to a **cache**. *Collect* writes; *Consume* reads. That turns
Butler into *"a Generic Read API to any of the data sources that the Butler
collects from"*.

The documented pattern: a **high-frequency internal duty collecting**, and a
**second manually-triggered duty consuming**, called from outside the firewall —
so an external service never touches the internal network.

### Named modules

**Collectors:** Database Reader · Duty cache consumer · EventHub Monitor ·
Folder monitor · iSeries Monitor · M3 Autojob status · M3BE Monitor · M3Grid ·
M3Grid – Users logged on · MEC In-Channel Monitor · Server Running.

**Publishers:** Call REST API · Database Writer · Disk Writer · Grid Application
Start/Stopper · M3BE Job Starter · MEC ReProcessor · Send Email · SMS Sender ·
Statistics Writer.

**Converters:** JSON-converter. **Listeners:** Event Listener.

### Dynamic parameters

Usable in **any text-field parameter of any publisher**:

`$$_SUM(col)_$$` · `$$_TABLE()_$$` · `$$_NUMROWS()_$$` · `$$_ALL(col)_$$` ·
`$$_FIRST(col)_$$` · `$$_CURRENT(col)_$$` · `$$_LAST(col)_$$` · `$$_DUTY()_$$` ·
`$$_DATE()_$$` · `$$_TIME()_$$` · `$$_DATETIME()_$$` · `$$_TIMESTAMP()_$$`

### Other vocabulary

Dashboards (12-column grid, unlimited rows, drag-resize **widgets**, per-group
read and/or write access) · Statistics definitions · Service windows · Groups ·
Users · Monitor Tokens · Connections · Offset scheduling · Export & Import
duties.

## Architecture and installation

A **host application plus several module applications**, all in a **servlet
container**. Host and modules may sit on different servers, data centres or
countries provided the host can reach the modules over HTTP.

**Recommended:** host and module in **separate Tomcat servers**, same or
different Windows/Linux box — the security argument being that module access can
then be blocked at the firewall. **Butler itself needs no internet access.**
The UI is browser-based, any browser or device.

**Installation** (in a technical space separate from the product tree):
**Tomcat 8.5 64-bit** · **AWS Corretto 8 64-bit JRE** (Java 8) · `vbuinstall.zip`
· SQL Server Management Studio · an **NTLM auth DLL** copied into Java's `/bin`
· an SSO DLL. Database named **`vincebutler`**, created and populated with
`fullscript.sql`; a SQL service user granted access; `vb-host.properties` (and
optionally `server-host.xml` / `server-modules.xml`) edited; `vbuinstall.ps1`
run from an elevated PowerShell; the VBU Host service set to run as the VBU
service user.

**Pre-install checklist:** customer network access · VBU DNS hostname · Windows
and SQL service-user credentials · install files on or uploadable to the server ·
**service-user passwords that do not expire** · the webroot-vs-`vb-host`
decision · port · HTTP/HTTPS + certificate · self-signed root CA.

**Encryption** (from v1.3.x): `encryption.secret.location=NONE/STATIC/FILE` —
**FILE recommended**; `encryption.secret.path` (the folder must be created
manually); after generation, tighten `vbu-secret.properties` permissions so only
the service account has Full Control (disable inheritance, remove all inherited
permissions).

**LDAP** is configured by a documented **temporary** procedure: add the LDAP
config to `vb-host.properties`, start, verify the row appears in
`dbo.connection`, stop, then **remove the LDAP configuration from the properties
file**.

**Connection types:** Database · HTTP server · SMTP server · FTP server · File
server. Database Reader supports PostgreSQL, MSSQL, Oracle, DB2, MySQL and
MariaDB, with more addable on request to support@vince.no.

## Gotchas — several are genuinely dangerous

- > **"WARNING: if the encryption key is stored in a file, and the file gets
  > deleted, all connection passwords will be unrecoverable."**
- **`encryption.useDpapi` is flagged "Currently not working"** in the
  installation guide.
- **Statistics: maximum 20 fields of the same type.**
- **All publishers in a duty receive the same input** — the last output from the
  latest Collector in that duty. **You cannot fan different data to different
  publishers within one duty.** This shapes how you design duties.
- **`$$_CURRENT()_$$` only works on publishers that have a "Run for each row"
  parameter**; elsewhere it **silently degrades to `$$_FIRST()_$$`**.
- **Disk Writer** *"assumes that the service user running Tomcat can write to
  this folder"* — no stated check or error path.
- The **"Issues & Fixes" page is close to empty** — an untitled "Error:"
  heading, an empty embed, and two children.

## Key pages

[Duties](https://app.notion.com/p/1be45c018bbb4a05902a32f7064579cb) ·
[Dashboards](https://app.notion.com/p/b91e6c5c56f241509991fb996a915361) ·
[Administrator tab](https://app.notion.com/p/964428417388473282e36f3a69a20595) ·
[Set your own start page](https://app.notion.com/p/7113bfcd48824b76a0deb1f90ab07608) ·
[How-to collection](https://app.notion.com/p/6980eb5f1e254b39b16ebf04e565cdb6) —
naming convention; publish result as CSV on email; link from VBU to a
SmartOffice program; use data from different sources in one result set; use REST
API to update M3; send data via API to a VBU duty ·
[VBU Guidelines](https://app.notion.com/p/71f89da69d404ce2885f1e56cb50aa19) —
design advice for new duties, dashboards, widgets ·
[Install checklist](https://app.notion.com/p/571ae2eb20964b1eb54c121f53973898) ·
[Install guide](https://app.notion.com/p/460a61d60d534494ab7b93d62a8eb2f6) ·
[VBU technical root](https://app.notion.com/p/da58d13242814972a9b126fe3d42d332)

## Lifecycle status — unstated

**No deprecation notice and no successor named.** Signals only: user
documentation is largely 2023-era; the installation guide was edited Sep 2025;
Guidelines Jun 2026; Vince Butler appears in the 2026 Professional Services
Price List and in the internal skill matrix ("VBU install", "VBU duties"). Net
Change Reports run 1.23 → 1.28.

## What the documentation does NOT answer

- **The authoritative user guide is apparently the external Confluence wiki**,
  not Notion (see Provenance).
- **No supported-version matrix**: which M3 releases, which MEC/EventHub/Grid
  versions, which OS. **Tomcat 8.5 and Java 8 are both long past their prime**
  and a "Java Migration Guide" page title implies movement — check it before
  quoting the stack.
- **No sizing guidance** (CPU/RAM/disk) — stated for VSE, absent for VBU.
- **No duty-failure semantics**: retries, alerting on a failed duty, what
  happens when a Collector times out, or cache retention/eviction.
- **No stated authentication model for the product itself.** LDAP is described
  as a bootstrap step; SSO appears only as an unexplained "SSO DLL" in the
  checklist.
- **Per-module parameter reference** beyond Database Reader and Disk Writer is
  not covered in Notion.
- **No API reference** for "send data via API to a VBU duty", or for Monitor
  Tokens.
