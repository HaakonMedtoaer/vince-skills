---
name: vince-web-apps-and-maintenance
description: Use for Vince's bespoke web applications and customer unique solutions (H5 custom solutions, VIO, microservices such as Protan Project Center, Plantasjen CampaignEngine/PriceEngine, Systemair Mini ERP, Riis Budget App), and for the Maintenance Request process that supports them. Covers the decisive question of whether an incoming request is covered maintenance or must become a billable Change Order, the response-time frames, and where a given customer solution is documented.
---

# Vince web applications, customer unique solutions and maintenance

The bespoke side of the business — apps built for one customer — and the
process that keeps them alive afterwards.

## Provenance

Compiled from Vince's Notion workspace on **2026-09-22**. Notion is the source
of truth. Be aware that the register described below is **uneven in quality by
design accident, not by intent** — see *Documentation quality* — so always open
the customer's own page before telling anyone what exists.

## The single most important rule

**Maintenance covers keeping an existing feature, service or app functioning.
Any request for functionality that was not present at handover is not
maintenance — it becomes a Change Order.**

The typical genuine maintenance case: **the customer's M3 was updated or
changed and that broke the Vince product.**

Two corollaries that decide real tickets:

- Products must be functional **before** official handover, so a
  not-yet-delivered solution is out of maintenance scope entirely.
- **If there is no maintenance agreement, the request is converted to a Change
  Order.** The process page supplies ready boilerplate e-mail text for exactly
  this conversion — use it rather than improvising.

## What this line of business is

Bespoke web applications and microservices for individual customers, mostly
around Infor M3:

- **H5 Web Application Custom Solutions** — embedded in M3's H5 client
- **VIO** — appears across several customers
- Standalone apps and microservices

Named solutions on the register:

| Customer | Solutions |
|---|---|
| Optimera | H5 Printer Setup, H5 Kontroll før fakturering, H5 Batchordre, H5 Sales Overview, H5 Item Search, VIO, VPR, Initial load |
| Flisekompaniet | VIO |
| Brødrene Dahl AS – SGDN | Invoice control, Initial load |
| Plantasjen Norge AS | Pick & Pack Service & Maintenance, CampaignEngine, PriceEngine, Floriday Integration, Fotoware, POS integration, Customer microservice, AWS Content Hosting |
| Systemair AB | Mini ERP |
| Protan | Project Center |
| Pfaudler GmbH | VL custom application Item Search |
| Varner AS | VIO |
| Lantmännen Maskin AB | VIO – EQM |
| Riis Bilglassgruppen | Budget Application |

## Documentation quality — set expectations before you look

The Web Applications page is a **register, not a guide**: a table of Customer /
Description / Documentation link / Agreement / Documentation owner. Quality
varies enormously, and knowing this saves a wasted hour:

- **Systemair Mini ERP is the fullest** — ~14 child pages: user manuals (Pick
  and Pack, Sales Overview, Item Search), *Technical Documentation – Vince Apps
  – Systemair*, MeC Mapping Details for TRN, IDM Application, API LIST, CMS100MI
  Transactions, Programs Used for H5 Scripts, H5Script/Personalize notes, Web
  Apps Installation guide, User Logs, plus a requirements/setup .docx for M3 CE.
  Use it as the model of what good looks like.
- **Riis Budget App** shows the common pattern: agreement and project document
  as attached PDFs, changes as dated child pages.
- **Optimera Sales Overview** is two links.
- **Protan Project Center and Plantasjen Customer microservice are completely
  blank pages**, despite being listed.

The Change Order process says completed-work documentation goes into Notion
(no removal policy), and the Maintenance process points developers at this same
register for prior technical documentation — so this is the intended single home
for it, whatever its current state.

## The Maintenance Request process (6.5)

**Applies only when** there is a contract **and** the customer pays a
maintenance fee, the ticket genuinely qualifies as maintenance, and **the ticket
is registered in the Customer Portal**.

**Roles:** Maintenance Team Lead · Developers · Queue Manager.
**Tools:** Moment (PSA) · Notion (documentation) · HubSpot (customer requests).

**Steps:**

1. Customer registers the request in the Customer Portal.
2. Verify it really is a maintenance request (apply the rule at the top).
3. Check the *Current Customers with Maintenance Contracts* list.
4. Reply in the portal. **The process page supplies full boilerplate e-mail text
   for every branch** — acknowledgment, conversion-to-change-order, estimate,
   resolution, revised timeline. Use it.
5. Create the ticket in Moment.
6. Team Lead assigns a developer, who writes story points and estimates.
7. Developer sends estimated time and scope back through the portal.
8. Do the work. **If the estimate exceeds 1 week, update the customer weekly.**
9. Deploy and confirm — or warn as early as possible if the estimate will be
   missed.
10. **Write technical documentation and update the existing documentation
    folder** (on the Web Applications register).
11. Send the satisfaction survey; close tasks in Moment.

**Response frames:** first contact **6 h** · delegate and start looking **6 h** ·
give estimate or status **12 h**.
**KPIs:** satisfaction survey, response time (SLA), resolution time, new
tickets/month.

## Agreements and contracts

The register's Agreement column marks an X for: Optimera Sales Overview,
Optimera Item Search, Brødrene Dahl Invoice control, Plantasjen CampaignEngine,
PriceEngine and AWS Content Hosting, Systemair Mini ERP, Pfaudler Item Search,
Varner VIO, Lantmännen VIO-EQM, Riis Budget App.

The maintenance process holds the authoritative list of **maintenance**
contracts with yearly ARR — 12 solutions, roughly 1.74 MNOK total, all flagged
Custom Development. **These two lists do not reconcile** (Dahl OY H5 Sales
Overview appears in the maintenance list but not on the web-apps register), and
nothing states which is authoritative. Check both, and flag the discrepancy
rather than picking one.

## Key pages

- [Web Applications / Customer Unique Solutions](https://app.notion.com/p/763bab5b8bf84d719d7a015f964b61c7)
- [6.5 Maintenance Request Management](https://app.notion.com/p/dd56af6be734438e938fde254b59633c)
- [Systemair Mini ERP](https://app.notion.com/p/4df766f51de641899b7bf1d27663a01e) — best-documented example
- [Riis Budget App](https://app.notion.com/p/2b88e766df5380f8b01ce19a26087b62)

## What the documentation does NOT answer

- **No definition of the line of business itself** — how such a project is sold,
  scoped, estimated or handed over. The register assumes you already know.
- **No documentation standard or template.** No required sections, and no rule
  about where agreements live (some are Notion attachments; the Change Order
  process says signed CR documents go to the customer folder on the share).
- **VIO, VPR and "H5 Web Application Custom Solution" are never defined** as
  terms anywhere.
- **Process page defects**: step 5 of the first numbered list is truncated, the
  survey tool/method is explicitly "not selected: WIP", and the KPI table's
  location-of-results is TBD.
- **No hosting, environment, deployment or on-call model**; no security or
  data-processing terms; no versioning or end-of-life policy for these apps.
- **No stated link between a maintenance ARR figure and what hours or severity
  levels it buys.**
