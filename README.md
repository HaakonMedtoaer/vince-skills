# vince-skills

Claude Code skills for Vince AS consulting work — Vince products and Infor M3
business-area knowledge. Packaged as a plugin marketplace so the same set loads
on every machine, with git history behind it.

## Install on a machine

```
/plugin marketplace add HaakonMedtoaer/vince-skills
/plugin install vince@vince-skills
```

Run these from an interactive `claude` terminal. Afterwards the skills are
available in every project on that machine, invokable by name (`/vince-peppol`)
or fully qualified (`/vince:vince-peppol`).

To pick up changes later:

```
/plugin marketplace update vince-skills
```

## What's in here

One plugin, `vince`, holding 35 skills in three families.

**Vince products** (13) — written from the Vince Notion workspace, September 2026.
Every page cited; every gap the documentation leaves open is stated explicitly
rather than filled in.

`vince-live-platform` · `vince-live-administration` · `vince-live-rest-api` ·
`vxl-live-excel-addin` · `vince-dashboards-and-widgets` · `vxl-classic` ·
`vse-vince-security` · `vbu-vince-butler` · `vince-peppol` ·
`vince-app-builder` · `vince-customer-portal` · `vince-services-and-training` ·
`vince-web-apps-and-maintenance`

**Vince Live workflow steps** (16) — confirmed JSON shapes captured from real
customer workflows and real tenants, originally from the VinceGenerator project.

`vince-trigger-step` · `vince-m3-native-api-step` · `vince-generic-api-step` ·
`vince-transform-step` · `vince-generic-filter-step` · `vince-excel-step` ·
`vince-email-step` · `vince-sms-step` · `vince-converter-step` ·
`vince-data-lake-step` · `vince-table-updater-step` ·
`vince-exportmi-select-step` · `vince-native-vs-pipeline-decision` ·
`vince-field-metadata-lookup` · `vince-custom-tables-search-api` ·
`vince-app-builder-handoff`

**Infor M3 business areas** (6) — general M3 functional knowledge, explicitly
*not* tenant-confirmed. Program and transaction names must be checked against
the tenant's own catalog before use.

`m3-order-entry-ois` · `m3-purchasing-pps` · `m3-inventory-warehouse-mms-mws` ·
`m3-manufacturing-pms-mos` · `m3-financials-gls-aps-ars` ·
`m3-business-partner-crs`

## The rule these skills are written to

**Never present an unconfirmed guess as a Vince fact.** Each skill states where
its content came from, and ends by naming what the source does *not* answer.
Where two sources disagree, the disagreement is recorded rather than resolved by
picking one — see `vse-vince-security` on the two different things called "Vince
Security", and `vince-live-rest-api` on the Custom Tables search endpoint.

When editing: check the source, then change the skill — don't soften a stated
gap into a confident answer because it reads better.

## Related, not in here

- `vince-live-workflow` and `m3-sod-analysis` arrive separately through the
  account-synced pool. That sync is **download-only** — local edits to them are
  overwritten, so corrections have to be made at their source.
- The workflow-step skills also exist in `Projects/VinceGenerator/skills/`,
  which is not a `.claude/skills/` directory and does not load. This repo is the
  copy that ships; keep edits here.
