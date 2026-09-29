---
name: vince-email-step
description: Confirmed JSON shape of the Vince Live EMAIL workflow step (target workflow-email), including its Handlebars body templating. Use when a workflow should send a report, notification or reminder by email, when asked "how do I send an email from a workflow" or "would this EMAIL JSON work", or when a design routes mail through a REST call instead of this step.
---

# `EMAIL` step

`type: "EMAIL"`, `target: "workflow-email"`. Shape captured from real tenant workflows.

## Shape

```
{ "subject": …, "body": …, "isHTML": …, "to_addresses": [ … ], "attachmentSource": … }
```

## `body` is full Handlebars

- Values: triple braces, e.g. `{{{body.subject}}}`
- Conditionals: `{{#if body.summary}} … {{/if}}` (Homewerks)
- Loops: `{{#each body.summary}} … {{/each}}` (Homewerks)

Not JSONata (Transforms) and not the double-brace syntax of `GENERIC_FILTER`.

## Use this step for mail

If a design sends mail through a `GENERIC_API` call to an SMTP or mail-relay endpoint, redirect it
here — that pattern has no basis in any real workflow. A workflow that pulls data, shapes it and
emails it is a complete app on its own: it needs no Custom Table or screen.

## From Vince's product documentation (claims, not captures)

- **CC and BCC recipients don't receive the mail** — stated as a known issue.
- The attachment type must be **File** when attaching an Excel step's output and **Data** for M3
  Filter, REST API, Transform or Code output; choosing File for a non-Excel step makes the workflow
  fail at run time.

## Not known

- The allowed values of `attachmentSource`.
- Whether CC and BCC exist in the JSON at all, given the documented known issue.
