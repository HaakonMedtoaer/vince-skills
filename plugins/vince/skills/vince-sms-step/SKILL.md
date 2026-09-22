---
name: vince-sms-step
description: The confirmed (but thin) JSON shape of the Vince Live SMS workflow step (target workflow-sms). Use when drafting or reviewing a workflow that should send a text message notification, or when someone asks "how do I send an SMS from a workflow" or "would this SMS step JSON work" — and to be reminded this step type has less real-world confirmation than the others, so don't over-trust invented detail around it.
---

# SMS step

`type: "SMS"`, `target: "workflow-sms"`.

## Confirmed config shape

`definition.stepConfig[stepId]`:

```
{
  "receiver_numbers": [ ... ],
  "content": ...
}
```

That's the full confirmed shape — two fields. This is deliberately thin.

## This is the least-confirmed of the ten step types

This shape comes from `workflow with all the steps.txt`, a reference workflow built to hold one example of each of the ten confirmed step types — not from any real customer production workflow surveyed so far (unlike, say, `EMAIL` or `TABLE_UPDATER`, which have real customer-production confirmation on top of that reference file). Nothing is currently known about:

- Number formatting requirements (country code, international format, etc.)
- Character limits or truncation behavior on `content`
- Templating language used inside `content` (untested whether it's Handlebars, JSONata, or plain string substitution)
- Delivery/retry/failure behavior

Don't fill in any of the above from assumption when drafting or reviewing — flag them as open questions if a brief needs them, rather than presenting a guess as fact. This is exactly the kind of step where "a plausible-looking value is the failure mode this project exists to prevent."

## When to reach for this skill

- A brief asks for a text/SMS notification — use the two confirmed fields, and explicitly flag anything beyond them (formatting, templating, limits) as unconfirmed rather than inventing it.
