---
name: vince-live-administration
description: Use for administering Vince Live rather than building in it - creating a tenant, setting up the M3/ION connection or a OneDrive/SharePoint connection, on-prem ION reachability and static IPs to whitelist, API Clients and the client-credentials token flow, users, roles and ABAC permissions, tags, SSO via OpenID Connect (Entra/Okta), SCIM user provisioning, and the hosting/encryption/shared-responsibility security answers customers ask in questionnaires. Also use when a connection, login or permission is failing and you need the documented causes.
---

# Vince Live — administration, connections and security

The setup and operations side of the platform.

## Provenance

Compiled from Vince's Notion workspace on **2026-09-22**. Notion is the source
of truth. **None of the pages read are marked verified.** Two specific cautions:

- Some content is a Confluence→Notion migration residue. An archived
  *"Setup SSO (Customer)"* page describes a **SAML** app — that is superseded;
  SAML is not supported.
- Security-questionnaire answer pages are **sales-facing and looser** than the
  product documentation. Do not quote them as shipped behaviour.

## Tenants

A **tenant** is "a dedicated customer environment within the Vince platform" —
one customer, one tenant. It is the boundary for users, roles, connections,
environments, workflows, tags and custom tables.

**Creation is Vince-internal, not self-service.** A Closed Won deal in HubSpot
raises a ticket in the *Customer Tenant Setup* pipeline. Inputs: customer name,
tenant administrator, selected products (for Vince Live: App Builder /
Painkillers / Peppol).

T&O team tasks: create the tenant and register its name; assign **Super Admin**;
apply required configuration; register the customer's HubSpot ID in the **Tenant
Administration App**.

> **Documented order that matters:** the creator sets *themselves* as admin
> first, then adds the customer's administrator — so Vince can install
> Painkillers and configure settings without involving the customer.

Afterwards the customer receives tenant URL and credentials and **owns the M3
connection step themselves**, plus installing the Excel add-in; they confirm
back by email/support.

- **Painkillers install** (Tenant Management app in Vince's admin tenant)
  requires a **Bearer Token lifted from the customer tenant's browser DevTools →
  Network tab**, with "migrate design files" and "migrate groups" ticked. Only
  after at least one connection exists.
- **App Builder** additionally needs a **Claude API key**, obtained internally
  and sent encrypted from support@vincesoftware.com.
- **Tenant ID** is at *Menu ⇒ About*, or Company Settings → Information. Needed
  for the Cognito token URL, the SSO redirect URL and support tickets.
- **Environments** (Development, Testing, Staging, Production, Education
  predefined; custom allowed) live under Connections → Manage Environments.
  Mandatory: Name, **Environment ID in VXL Live**, Description. A connection is
  bound to an environment.

## Connections

### M3 via Infor ION API — the standard one

**Prerequisites:** Infor M3 BE **13.4+** (June 2018, with ExportMI); **M3 REST
API v2 endpoints enabled and published in ION** — if absent, the customer must
go to Infor or their ION partner; ION Gateway reachable from the internet; ION
version requirement is literally **"TBD"**. Browsers: Chrome, Edge.

**In M3/ION:** Infor ION API → **Authorized Apps** → "+" → type **Backend
service** → Save → **Download credentials** → choose **Create a service
account**, naming a system user with **ordinary M3 access and a non-expired
password policy**, plus certain IFS security rules verified in user management
security roles → download the credentials file.

**In Vince Live:** hamburger → Connection → New Connection → name, alias,
description → System **M3** → tick **"Use M3 user ID"** → auth type **OAuth** →
upload the credentials file → protocol **https** → select environment → Save →
**Validate Connection** from the ellipsis → optionally **Import APIs** (per
program, or Advanced = all).

**The run-as role question.** The service account needs permission to *run as*
other M3 users. In cloud that is **M3BE-ConfigAdmin**, which does not exist
on-prem; on-prem it is often **M3BE-FndAdmin**, verifiable in Infor ION Grid.
A screenshot says the system user needs "at least these three system roles" —
**the roles are only in the image, never in text.** Treat the canonical list as
undocumented and environment-dependent.

### ION service account (for ION Connection Points / BODs)

Infor OS → IFS (old UI: User Menu ⇒ User Management; new UI: Menu ⇒ OS ⇒
Security) → Service Accounts → "+" → describe → attach an IFS user with
permission to call ION APIs → save. **The credentials download is offered once
only.**

### Vince Live as an API inside ION (reverse direction)

Register Vince Live in ION *Available APIs* with an OpenAPI/Swagger file
supplied by Vince. Endpoint `https://api.vince.live`; Proxy Context `api`; Proxy
and Target Endpoint Security both **OAuth 2.0**; Token Endpoint
`https://{tenantId}.auth.eu-central-1.amazoncognito.com/oauth2/token`; Grant
Type **Client Credentials**; Client ID/Secret from a Vince Live API Client.
Documentation type must be *Swagger*.

### On-prem ION reachability

Only **two endpoints** need to be reachable — the **Infor Secure Token Service
(STS)** (usually the XI/OS server) and the **ION API Gateway** (usually the
Ming.le server). Nothing else in the network is contacted.

Three documented options: open firewall ports directly (simplest, largest attack
surface); a **reverse proxy** (Nginx/Apache — needs a host, routing rules, SSL
certs, public IP/DNS); or an **existing API Gateway**, which **must not wrap or
alter the ION response** (supported auth: OAuth 2.0, Basic, API key, AWS
Signature).

**Static outbound IPs** to whitelist, via the optional Vince Live proxy service:
`3.124.254.204`, `35.156.168.80`, `35.157.203.168`.

### OneDrive / SharePoint (Azure)

Needs an M365 admin with Cloud Application Administrator, Application Developer,
Application Administrator or Global Administrator.

The File Picker triggers admin consent for **delegated** scopes — enough for
**manual file selection only**. For automated/scheduled folder reads: Azure App
registration → client secret → Graph **Application** permission
**Sites.Selected** → grant a write role to the app **on each SharePoint site** →
in Vince Live a connection with System **Azure**, auth type "Auth", HTTPS,
environment, Client ID, Client Secret, Token URI (v2), Base Uri
`https://graph.microsoft.com/v1.0/`, grant type `client_credentials`, scope
`https://graph.microsoft.com/.default` → **Validate Connection**.

> **Delegated vs Application is the usual failure.** Delegated breaks automation.

### CONO / DIVI

Connection-level CONO (digits, optional) scopes data to one company. Blank with
"M3 User ID" ticked → the M3 user's default CONO. Neither → the service user's
CONO. V2 moves CONO/DIVI into the M3 API step with source "From Client" or
"Constant". **A connection-level CONO takes precedence over a workflow one.**

## Authentication and API access

- Interactive login: strong auth with **MFA**; SSO via **OpenID Connect**.
  **SAML is explicitly not supported.**
- **SSO setup** (shipped): admin → **Company** → Single Sign On → **Add Identity
  Provider** → **Microsoft** or **Okta** → name, **Issuer URL**, Client ID,
  Client Secret → Save. Issuer URL: Entra
  `https://login.microsoftonline.com/<Entra Tenant ID>/v2.0`; Okta
  `https://<company>.okta.com`. **Enforce SSO** is a toggle.
  Gotchas: the SSO option may not appear until the browser tab is refreshed or
  cache cleared; **Okta does not support PKCE**; for Entra record the client
  secret **Value**, not the Secret ID, and uncheck Access and ID tokens under
  Implicit grant.
- Redirect URL pattern:
  `https://<TENANT ID>.auth.eu-central-1.amazoncognito.com/oauth2/idpresponse`
- **API Clients** (machine-to-machine): Menu → API Clients → add → Manage Client
  reveals **Client ID / Client Secret**. **Client Credentials OAuth2 flow.**
  Token: POST to
  `https://{tenantId}.auth.eu-central-1.amazoncognito.com/oauth2/token`, Basic
  auth with Client ID/Secret as username/password,
  `content-type: application/x-www-form-urlencoded`, body
  `grant_type=client_credentials`. Use the access token as a **Bearer** token.
  *(The API Connection page prints a malformed host — `...amazonaws.com.amazoncognito.com`.
  The form above is the correct one.)*
- **An API Client with no Role assigned can do nothing.** Stated explicitly, and
  a common silent failure. Assign roles from the Security page or the API Client
  page.
- Documented API areas: Auto-increment, Common features, Custom Tables, Tenant,
  Tags, Utils, Schedules, Variables, Workflows, Connections ("To be Described").

## Users, roles and permissions

Two permission levels: **Admin** (unrestricted, needs no roles) and **Regular
User** (needs roles).

**User creation:** Add User → First Name, Last Name, Email, Permission (all
mandatory) → invite email with tenant name, email, **temporary password** and
link → forced password change at first login. **Email is immutable after
creation.** Users can be **deactivated**; deletion is not possible today.

**M3User ID** on the user: every M3 transaction then runs as that M3 user, so M3
API permissions apply and workflows fail if the user lacks them. Unset → the ION
service account, which is also used for all non-interactive workflows.

**Tags** (key/value, per user) drive the workflow input source "From Tag" and
dashboard filters (`{{ user.tags.X }}`). Limits: one value per tag; tag names
cannot be renamed (renaming creates a new tag); **tag-value changes need a log
out/in before dashboards refresh**.

**Authorization is ABAC/RBAC.** A permission is **App / Resource / Resource Name
/ Actions**:

- **Apps**: Foundation, Custom Tables
- **Resources**: API Clients, Connections, Dashboard, Environment, Gateway,
  Group, Meta data, Role, User, Variable, Webhook, Workflow; Custom Tables →
  Data and Configuration
- **Actions**: `*`, Read (execution only), Write (execute + edit). Resource Name
  `*` = all instances.
- **Deny rules are supported** alongside allow rules.
- Roles apply only to "User"-permission accounts.

Role lifecycle: create (unique name) → modify permissions (name is locked) →
**Deactivate** (removes from all users, revocable) → **Delete** (permanent).

> **The rule that explains most "workflow won't run" tickets:** to execute *any*
> workflow, a regular user must have access to **Connections, Environments and
> Metadata.**

Also: a role linked while creating or modifying a resource is added with
**Read** access, and **copying a resource does not copy its role links**.

## SCIM provisioning — shipped

Admin → **Company Settings → SCIM** → pick Microsoft or Okta → Generate token
(**shown once**) + SCIM Base URL. Supports `GET/POST /Users`,
`GET/PATCH/PUT/DELETE /Users/{id}`, `filter=userName eq "..."`. Attributes:
`userName`→Email (required, unique, lowercased), `name.givenName`,
`name.familyName`, `emails[0].value`, `active`.

- SCIM-provisioned users get **no invite email**; default role **TenantUser**.
- `active:false` deactivates; DELETE permanently removes.
- **Group provisioning is NOT supported** — a documented production bug shows
  Entra group assignment failing with a generic 404. Workaround: disable Group
  provisioning in the IdP.

**Explicitly backlog, NOT shipped** — do not promise these: creating or syncing
**roles** from Azure AD over SCIM; **M3 User ID over SCIM**; SCIM group
provisioning; deletion of never-active users; the internal Vince Live Admin
Portal. The questionnaire claim that "roles can be assigned based on IdP groups
or claims" is hedged and not backed by a shipped feature.

## Security model — the questionnaire answers

- **Shared Responsibility Model**: AWS secures the cloud; **Vince** secures,
  configures, monitors and operates the platform within AWS; **the customer**
  secures their data and **user access** within Vince Live. No per-control
  responsibility matrix is published beyond this.
- **Encryption**: AES-256 at rest, customer-specific keys "whenever possible",
  keys in **FIPS 140-2** HSMs via AWS KMS; **TLS 1.2+** in transit, earlier
  versions unsupported.
- **Hosting**: AWS **eu-central-1 (Frankfurt)** for European customers,
  serverless, multi-AZ where the underlying services support it.
- **Backup**: DynamoDB **PITR, 35-day window**; S3 versioning with lifecycle
  retention tags of **14 / 30 / 90 days**. Restores are performed by Vince.
- **Allow-listing**: production `*.vince.live` HTTPS/443 and `graphql.vince.live`
  **WSS**/443 — WSS is required, there is no long-polling fallback.
- **Patching** is Vince's, as part of the managed service; customers install
  nothing. VXL Live updates itself and talks only to Vince Live. Deployment via
  Microsoft AppSource or M365 Centralized Deployment; no client admin rights.

## Documented failure causes — check these first

| Symptom | Documented cause |
|---|---|
| VXL Live simply doesn't work | M3 **REST API v2 endpoints missing in ION** |
| Run-as fails | service account lacks run-as; role differs cloud vs on-prem |
| **"Invalid user"** on an M3 API test | **M3 caps usernames at 10 characters**; Infor OS IFS does not |
| Connection breaks later | service-account password policy **expired** |
| Lost ION credentials | the file downloads **once only** — recreate |
| On-prem config upload fails | generated domains/ports are **internal**; hand-edit to public |
| ION responses malformed | an API Gateway in front of ION is **modifying the response** |
| API Client does nothing, no error | **no role assigned** |
| SCIM 401 / duplicate user / not syncing | invalid-expired token / user created manually first / provisioning off in IdP |
| SSO option missing | browser refresh or cache clear needed |
| OneDrive automation fails | **Delegated** chosen instead of **Application** permission |
| Dashboard ignores new tag value | user must log out and back in |

## Key pages

[Setup various Connections](https://app.notion.com/p/ef5bba4d1ec449ccbe1d238d02ee5972) ·
[Connection (on-prem options)](https://app.notion.com/p/d4724b8e921944e4a28eb0e253d34369) ·
[Q&A (M3 connection)](https://app.notion.com/p/93d218e88e0b48288b652b7581c1b170) ·
[Setup OneDrive Connection](https://app.notion.com/p/78a8fe7b9ff04c03a6663d9241404199) ·
[Environment](https://app.notion.com/p/f72c25fc790b4344986be83a868afb6e) ·
[API Connection to Vince Live](https://app.notion.com/p/6948c14475534ca9ba5c94796ab111d1) ·
[Connecting ION to Vince Live](https://app.notion.com/p/13d8e766df5380978772e0f930118c75) ·
[Configure Service Account](https://app.notion.com/p/13d8e766df5380d880fdf4693b1e8428) ·
[SSO Setup](https://app.notion.com/p/44961f4beb034a44ac31b446c3320363) ·
[Setup for Microsoft Entra](https://app.notion.com/p/f158988d5f6941f98b7125e66f23e71e) ·
[Setup for Okta](https://app.notion.com/p/59eadd1e9fbb4bab9ef6bafba181d27d) ·
[User Provisioning (SCIM)](https://app.notion.com/p/1528e766df53804b996be5f0d8d74422) ·
[User Management](https://app.notion.com/p/1f18e766df5380018c85dc873101a07a) ·
[Role management](https://app.notion.com/p/1548e766df538058b6eae99380f77f68) ·
[Roles and ABAC](https://app.notion.com/p/e1f31b710afc4c0c9fd4d33af76387c3) ·
[Tag Management](https://app.notion.com/p/8d5d3355b7f54ba79465a7bcff04ce89) ·
[Vince Live Security Overview](https://app.notion.com/p/f8cbd5a48ace434b907a857108888fe2) ·
[Static IP for on-prem traffic](https://app.notion.com/p/29dcd1b3904947dbbb6f5257ac7e6469) ·
[6.3.5 Customer Tenant Setup Process](https://app.notion.com/p/3228e766df53805a9289e23e560ca07f)

## What the documentation does NOT answer

Be explicit about these rather than improvising — several are questions
customers ask in security reviews:

- **The three required M3 system roles exist only inside a screenshot.** No
  textual list, no canonical per-environment answer.
- **Token lifetimes and refresh behaviour** — nothing states how long a Cognito
  or API Client token lives, or whether refresh tokens are issued.
- **No way to rotate or revoke an API Client secret** is documented, and no
  secret expiry is stated.
- **ION version requirement is literally "TBD".**
- **No SSO troubleshooting depth**: claim mapping, JIT provisioning, what
  happens to existing local users when Enforce SSO is switched on, or
  break-glass access if the IdP breaks.
- **Role semantics are under-specified**: how deny resolves against allow,
  whether permissions are additive across roles, and what `Gateway`, `Meta data`
  and `Variable` actually gate.
- **Users cannot be deleted**, and there is **no published offboarding or
  data-deletion procedure** for a departing customer.
- **No audit-log or access-review documentation** for a customer admin.
- **Nothing on multi-environment promotion** — how a connection or workflow
  moves Dev→Prod, or whether separate tenants are used instead.
