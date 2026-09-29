---
name: vince-sms-step
description: The confirmed but thin JSON shape of the Vince Live SMS workflow step (target workflow-sms). Use when a workflow should send a text-message notification, or when asked "how do I send an SMS from a workflow" or "would this SMS JSON work" — and to keep unconfirmed details out of the design.
---

# `SMS` step

`type: "SMS"`, `target: "workflow-sms"`.

## Shape

```
{ "receiver_numbers": [ … ], "content": … }
```

That's all that's known. The only evidence is a captured reference workflow built to hold one of each
step type; no live customer workflow using SMS has been seen, and Vince's product documentation page
for it is empty.

## Unknown — flag, don't fill in

- number format (country code, international form)
- length limits or truncation of `content`
- which templating language `content` uses
- delivery, retry and failure behaviour

If a brief needs any of these, list them as open questions in the design.
