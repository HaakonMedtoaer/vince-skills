---
name: vince-app-builder
description: Use for the Vince Platform App Builder - generating business apps from plain-language prompts against Vince Live workflows and Custom Tables. Covers prompting conventions and resource aliases, the run-a-workflow-3-times rule before adding it as a resource, when you must re-prompt, the version lifecycle (build session, save, discard, new version, publish, archive), publishing and rollback, and the two separate role systems (Vince Live roles Read/Run/Write/Deploy/All versus per-app InApp Roles). Also use for App Builder access rights, external users, or the Claude API key a tenant needs.
---

# Vince Platform App Builder

Generating business apps from plain-language prompts against workflows and
tables that already exist in Vince.

## Provenance and status

Compiled from Vince's Notion workspace on **2026-09-22**.

> **The Best Practice Manual is explicitly Draft v0.1 for review** (2026-09-17),
> derived from Vince's published docs at
> vincesoftware.com/product-documentation/app-builder, and it closes by telling
> the reader to **verify against the live docs before treating any mechanic as
> final**. Several sections — naming conventions, review gates, who owns Deploy
> — are deliberately left blank for the team to fill in. Carry that caveat into
> any answer.

## What it is and where it sits

App Builder is **"a capability in Vince Platform that generates business apps
from plain-language prompts against workflows and tables already in Vince."**

It is not a separate silo:

- It consumes **Vince Live workflows and Custom Tables as resources.**
- Apps run in the Vince host, under a tenant URL such as `.../ai-apps`.
- Access is governed by ordinary Vince Live roles, and **every request from a
  running app — internal or external user — still goes through Vince Platform's
  role and permission checks.**

**Commercially**, it is a per-tenant purchasable product. The Customer Tenant
Setup process lists it alongside Painkillers and Peppol at deal close, and adds
one hard operational fact: **a tenant with App Builder needs an API key to
Claude**, obtained internally and sent encrypted from the support mailbox.
Internally the key is a Connection in the tenant with system `Claude API`, and
**the user building the app must have permission to that connection to decrypt
it.**

## Prompting

Three habits:

1. **Be specific.** *"Build a dashboard from `invoice_table`, grouped by status,
   with counts for failed/sent/pending"* beats *"build a dashboard"*.
2. **Reference resources by alias.** Set aliases on workflows and tables
   *before* prompting, and use them in every prompt.
3. **Name the fields** you care about, or the AI may guess wrong.

**Iterate, don't rewrite.** Start with the simplest version → **Save** to lock
it in → add one feature per prompt → **create a new version** before a bigger
change. When changing something that exists, describe *what to change* ("move
the chart to the right side"), not the whole app again.

Five prompt templates are provided as starting points — Dashboard,
Search→Detail, List with filters, Form, Tracker — each parameterised on
`[table]` / `[field]` / `[workflow]`.

### The one genuinely non-obvious mechanic

**Workflows have no fixed schema. App Builder infers a workflow's output shape
from past executions.**

So before adding a new workflow as a resource:

1. **Run it at least 3 times** with realistic inputs.
2. Open the logs and confirm the final step's output has the fields you expect.
3. *Then* add it.

Skip this and App Builder either cannot use the workflow at all, or binds to
fields that do not exist.

### Re-prompt triggers — nothing updates automatically

Re-prompt when: a new resource was added and should be used · a field was added
or renamed in a table the app uses · a workflow's output changed (after
re-running it 3+ times) · a resource was removed.

Consequences of not re-prompting: added fields don't appear; renamed fields
cause runtime errors; a deleted resource breaks the app at runtime; a brand-new
workflow can't be used until it has been run a few times.

### Don'ts, stated as such

- **Never put secrets in prompts** — passwords, API keys, customer-identifying
  data. Prompts go to the AI as text.
- **Don't expect identical output from identical prompts.** The AI is not
  deterministic; iterate rather than retry blindly.
- **Don't write one giant all-in-one prompt** — long prompts give inconsistent
  results.
- **Don't delete a resource a published app depends on** — update or replace it
  first.
- **Don't forget to Save** — idle build sessions can expire, and unsaved work
  lives only in a cache.

## Versioning

The lifecycle, in order:

| Stage | Meaning |
|---|---|
| **Build session** | the active editing context — **only one user per app at a time** |
| **Save** | commits the draft into the version being worked on, and **ends the build session** |
| **Discard** | throws away unsaved changes, reverts to last saved |
| **Create new version** | copies current state into a fresh draft |
| **Publish** | promotes a version to live |
| **Archive** | removes a non-published version — **you cannot archive the only published version** |

Suggested team convention (explicitly "adjust as needed"): one version = one
meaningful change or feature batch, not one prompt; always create a new version
before work that could break the published app; save at every stable checkpoint;
keep a changelog note.

*Implementation background from internal MVP/Beta notes:* versions are
controlled via an App API, all iterations are stored in S3 after each
generation, a user can revert to any version, and old versions are deleted by an
S3 lifecycle policy after X days. Apps go Draft → Deployed with admin approval,
and **an app is only visible to its creator and admins until Deployed.**

## Releasing and publishing

> **Only one published version exists per app at a time — publishing a new
> version automatically archives the previously published one.** There is **no
> staged rollout**; publish is all-or-nothing for the tenant's users of that app.

Recommended flow: build and iterate → Save → **internal review/UAT with signoff
by both the end user and the system architect** → Publish. For the next change,
create a new version, iterate, save, publish.

**Pre-publish checklist:**

1. Re-prompted after any resource or schema change since the last publish.
2. Workflow(s) run 3+ times and output verified, if new or changed.
3. App Bridge logs checked for errors during testing.
4. Tested as **each relevant InApp Role**.
5. No secrets or customer-identifying data introduced via prompts or the tenant
   prompt.
6. Confirmed no dependency was deleted or renamed without a re-prompt.

**Rollback is the manual's weakest point, by its own admission:** republish the
last-known-good version if it was not archived and is still accessible, or fix
forward with a new version — and *"confirm exact rollback mechanics/permissions
with an admin, since a bad publish immediately archives the prior live
version."* Do not state rollback behaviour more confidently than that.

## Access rights — two separate role systems

Keeping these distinct avoids most of the confusion:

### Vince Live role — can you use App Builder at all

Set by the **tenant administrator**; scope is the **whole tenant**.

| Level | Recommended for |
|---|---|
| **Read** | viewers — managers wanting visibility |
| **Run** | end users — most people |
| **Write** | the small team actually building apps |
| **Deploy** | app owners — deliberately small |
| **All** | tenant administrators only |

### InApp Role — what you can see and do inside one app

Set by the app builder in the App Builder editor; scope is a **single app**.
Example: an "external" role that only sees its own records vs an "internal" role
that sees everything.

How it works: attach Vince Live roles or specific users to each InApp Role,
**tell the AI in the prompt how each role should behave** (*"for the 'external'
role, hide the cost column and disable the edit button"*), and **always test by
signing in as a user with that role before publishing.**

**InApp Roles never grant access to App Builder itself.**

**External users** (suppliers, transporters) can use App Builder apps: they sign
in with their own Vince Live credentials, are typically given **Run-only** Vince
Live roles by the tenant admin, and see only what InApp Role logic plus the
underlying Vince Live permissions allow.

**Least privilege:** grant the lowest level that works; assign roles to
**groups, not individuals**; keep Deploy small; review Write/Deploy holders
regularly (e.g. quarterly); remove access immediately on offboarding.
**Common pitfalls:** granting Write/Deploy too widely "just in case"; forgetting
to revoke on offboarding; putting confidential data in a **tenant prompt**
(brand colours fine, customer lists not); one blanket role for everyone;
skipping a sign-in test as the affected role after a role change.

Internally, permissions are modelled as `vrn` strings — e.g.
`vrn:TENANT-123:app-builder:*:*` at tenant level,
`vrn:TENANT-123:app-builder:app:Supplier Portal` action `RUN`,
`vrn:TENANT-123:app-builder:editor:Supplier Portal` action `WRITE`.

## Auditability and secrets

- **App Bridge logs** (in the App Builder editor) show every request/response
  between the running app and the Vince host — the debugging tool for one app.
- **Vince Platform audit logs** give the tenant-wide trail of role and
  configuration changes.
- Secrets: never in a prompt; never in the **tenant prompt** (brand and
  convention context only); **each environment (dev/test/prod) should have its
  own API key, never shared**; authentication to external systems belongs **in a
  workflow with proper connection settings, not in the app.**

## Related skill: deciding whether an app needs tables or screens at all

`vince-app-builder-handoff` covers the design-time handoff — including the point
that **Custom Tables and App Builder screens are optional, not required**: a
workflow can be a pure pull → transform → email report with no persisted state
and no UI, and that is a first-class outcome rather than an incomplete design.
It also warns against conflating `TABLE_UPDATER`'s step-level `command` field
with a table meta's `updateType`, and flags which claims in
`vince-app-generator-handoff.md` have been superseded by real captured workflow
evidence.

## How an app relates to workflows and tables

Tables and workflows are *resources* the app is prompted against, referenced by
alias. The App Builder Onboarding page shows both patterns concretely:

- A **Custom Table app** — invoice heads + lines tables, a join, drill-down, a
  metrics page. Warning attached: this route prompts the obvious customer
  question of where the data came from.
- An **M3 API app** — item groups → items → editable description with save →
  base price fetch → price update, where **each M3 interaction is its own small
  workflow** (`App onboarding List Item Groups`, `List Items in Group`,
  `Update Items`, `GetBasePrice`, `UpdBasePrice`). **Recommended for M3
  customers.**

Start simple and build up. Dashboards are mentioned only as *a thing an app can
be* ("Build a dashboard showing data from `[table]`"), not as a separate Vince
Live artefact the app links to.

## Key pages

[Best Practice Manual (Draft v0.1)](https://app.notion.com/p/3de8e766df53803c9d16c9f6eb961196) ·
[App Builder Onboarding](https://app.notion.com/p/3348e766df538063b5e0f425d1f34773) ·
[App Builder (internal tech doc root)](https://app.notion.com/p/2a18e766df53803ea8adf96d20e92bdd) ·
[MVP / Beta (infrastructure, versioning, vrn permissions)](https://app.notion.com/p/2958e766df5380919054dc401c995d7a) ·
[Permissions (two proposed structures)](https://app.notion.com/p/2b88e766df538027930af44fd5aaa58f) ·
[6.3.5 Customer Tenant Setup (Claude API key step)](https://app.notion.com/p/3228e766df53805a9289e23e560ca07f)

## What the documentation does NOT answer

- **Rollback mechanics are unconfirmed** — the manual itself says to check with
  an admin.
- **Version labelling**: the manual does not know whether App Builder supports
  version labels ("otherwise keep an external change log").
- **No worked prompt-to-app example end to end.** The templates are one-liners;
  onboarding points at apps in the playground tenant and a PowerPoint on
  OneDrive rather than reproducing the prompts.
- **Nothing on cost or token consumption** of the Claude API key per tenant,
  quotas, or what happens when it runs out.
- **No app-to-app story** — master app / menu structure and cross-app
  interactions appear only as "Future / v2 ideas".
- **No documented limits**: app size, number of resources, response-time
  expectations, or behaviour when a workflow is slow.
- **No relationship defined between App Builder apps and Vince Live
  dashboards** — whether they coexist, replace each other, or can link.
- **No migration or promotion path between tenants** (dev→test→prod) for an app.
  The only environment guidance is "separate API keys per environment".
- **InApp Role enforcement is prompt-driven** — described as something you
  *describe to the AI* and then *test manually*. **The docs never say whether it
  is enforced server-side**, which is exactly what a consultant needs when
  answering a security question. Do not claim it is.
- The internal `vrn` permission design and the manual's
  Read/Run/Write/Deploy/All levels are **not reconciled anywhere**, and the
  internal page still presents two competing structures.
