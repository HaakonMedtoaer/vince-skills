---
name: vince-email-step
description: Confirmed JSON shape of the Vince Live EMAIL workflow step (target workflow-email), including its full Handlebars body templating (triple-brace values, {{#if}}/{{#each}} block helpers). Use whenever drafting or reviewing a workflow that sends email, whenever someone asks "how do I send an email from a workflow" or "would this EMAIL JSON work," or whenever a brief describes a report, notification, or reminder that should land in someone's inbox — including report-only apps with no tables or screens at all.
---

# EMAIL step

`type: "EMAIL"`, `target: "workflow-email"`.

## Confirmed config shape

`definition.stepConfig[stepId]`:

```
{
  "subject": ...,
  "body": ...,
  "isHTML": ...,
  "to_addresses": [ ... ],
  "attachmentSource": ...
}
```

## `body` is full Handlebars, not just value interpolation

Confirmed patterns, from real customer workflows:

- Triple-brace value interpolation: `{{{body.subject}}}`
- Block helper `{{#if body.summary}} ... {{/if}}` (Homewerks)
- Block helper `{{#each body.summary}} ... {{/each}}` (Homewerks)

This is a genuinely different templating language from `GENERIC_FILTER`'s double-brace and `TRANSFORMER_MORPH`'s JSONata (see `vince-generic-filter-step` and `vince-transform-step`) — three languages, one workflow. When drafting an `EMAIL` step body, reach for real Handlebars constructs (conditionals, loops over arrays) rather than assuming it's a flat find-and-replace on top-level fields.

## Worth checking on every new draft: don't invent a REST mail connector

Before `EMAIL` was confirmed as a real step type, the generator invented a REST-based mail connector on two unrelated briefs — a fabricated shape with no basis in any capture. If you're reviewing generator output (or any draft) that proposes sending mail through a `GENERIC_API` call to some SMTP/mail-relay endpoint instead of an `EMAIL` step, that's the same failure mode recurring — flag it and redirect to `EMAIL`.

## An app need not have tables or screens

A workflow that pulls M3 data, transforms it, renders HTML, and emails it is a complete, valid Vince Live app on its own — a report-only app. Don't assume every workflow needs a Custom Table or an App Builder screen at the end; "pull → transform → EMAIL" is a real, confirmed shape (Topro's MMS090 export is one example, minus the email step which there goes to Excel — see `vince-excel-step` — but the same "no tables/screens needed" principle applies).

## When to reach for this skill

- Drafting or reviewing any notification, reminder, or report-delivery step.
- A draft proposes a mail connector that isn't `EMAIL` — flag it.
- A brief describes a report with no obvious table/screen need — confirm that's fine, not a gap.
