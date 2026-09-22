---
name: vince-customer-portal
description: "Use when a question involves the Vince Customer Portal, support tickets, or how customer requests are routed and owned — e.g. \"how does a customer raise a ticket\", \"which pipeline does this go to\", \"who is the queue manager\", \"the customer says they can't see their ticket\", \"is this a Change Order or a Maintenance request\", \"how do CaaS tasks get registered\", or anything mentioning the customer portal, HubSpot Service, ticket pipelines, or the Ticket Transit Line."
---

# Vince Customer Portal — tickets, pipelines and ownership

How customers reach Vince, how their request is routed, and what a consultant
is expected to do with it once it lands.

## Provenance and how to use this skill

Compiled from Vince's Notion workspace on **2026-09-22**. Notion is the source
of truth — this file is a fast lookup, not an authority. Before quoting
anything operational to a customer, open the linked page and confirm it still
says what this file says.

**The product-documentation page for the portal is effectively a stub.** A
consultant who looks only under *Vince Product Documentation → Customer Portal*
will find almost nothing. The real substance lives in the Process Library
(6.1.1, 5.3.1, 5.3.3) and in the internal HubSpot guide. That is the single
most useful thing to know about this topic.

## What the portal actually is

- A web portal at `https://vincesoftware.com/customer-portal` where customers
  register and track tickets.
- **It is a front end onto HubSpot.** HubSpot's support module is called
  "Service". Consultants work the customer's request *in HubSpot*, not in the
  portal.
- Three distinct systems are involved and should not be confused:
  **Customer Portal** (what the customer sees) → **HubSpot** (where tickets and
  conversations live) → **Moment** (the PSA tool holding time and projects).

## The routing rule that explains most confusion

**The ticket's pipeline is set automatically based on the first form the
customer fills out in the portal.** That pipeline determines the linked
process, the Process Owner and the Queue Manager. So "why did this land with
the wrong team" is almost always "the customer picked the wrong form", not a
routing bug.

Named pipelines (6.1.1):

| Pipeline | Has a linked process |
|---|---|
| Change Order | yes |
| CaaS Request | yes |
| Incident Management | yes |
| Questions Management | yes |
| Product Enhancement Request | yes |
| Maintenance Request | yes |
| Product Bugs | yes |
| Onboarding Pipeline | yes |
| Sales Leads | yes |
| Ticket Transit Line | **no linked process** |
| Internal Product Questions | **no linked process** |

Each pipeline has a Process Owner and a Queue Manager — look them up in 6.1.1
rather than guessing who owns a queue.

Note the guidance customers are actually given: *"For regular support requests,
please select 'Change Order'."* That is why Change Order volume is high; it is
the documented default, not a misuse.

## What the customer experiences

1. Visit the portal → **Register** → sign up with a work e-mail.
2. Click the verification e-mail — **the docs explicitly warn to check junk**.
   A customer "stuck at registration" is usually here.
3. Log in. Default view is a ticket overview, sortable by Status.
4. **Create Ticket** in the top menu opens the form. Attachments can be added
   at the bottom of the form, or added to the ticket after submitting.
5. Replies arrive by e-mail, and **the customer can reply directly from their
   mail client** — they do not have to come back to the portal. Their reply
   still lands on the ticket.

Professional Services and training packages can also be **ordered directly in
the portal**. A training package bought this way enters as a Change Order,
pre-approved for payment.

## What the consultant does (internal side)

From *Internal User Guide: HubSpot Support*:

- Work in **Tickets** (board or list view; drag between stages) and in
  **Conversations**, which is where replies to the customer are written.
- If a ticket is unassigned, **make yourself the Owner**.
- Use **Comment** for private internal discussion — it is not visible to the
  customer. Replies in Conversations are.
- Close the ticket when solved.

## Ties into the delivery processes

**CaaS (5.3.1).** It is the **customer's** responsibility to register CaaS
tasks in the portal; it is the **consultant's** responsibility to follow up.
While CaaS hours remain in the active month, check the portal for CaaS tasks.
If the hours run out and tasks remain, **inform the CSM** — do not silently
keep working. Updates on a task go back into the portal.

**Change Orders (5.3.3).** A portal submission is one of the inputs. Two facts
worth carrying:
- Change Orders are **always billable**.
- They are **capped at 10 man-days**; more than 80 hours moves it to the
  Project process instead.
- **Tickets entered manually by anyone other than the Queue Manager are not
  counted in KPIs.** If you create a ticket on a customer's behalf, you have
  quietly removed it from the numbers.

**Maintenance.** Whether something is maintenance or a Change Order is a
contract question with a sharp boundary — see the `vince-web-apps-and-maintenance`
skill, which carries that test in full.

## Key pages

- [Customer Portal (product doc — stub)](https://app.notion.com/p/ab191eb3702b486e8074f6b747b83140)
- [How to create tickets for Consulting Assistance and Support Tickets](https://app.notion.com/p/7148cc33d3624ac8a60bc88cd8934e9e)
- [6.1.1 Customer Portal Overview – Pipelines and Ownerships](https://app.notion.com/p/5502b1eec91f4c9fb96c9b2acf3be378)
- [Internal User Guide: HubSpot Support](https://app.notion.com/p/e964d9d1670d43eea02466a6e71bee44)
- [5.3.3 Change Order Management Process](https://app.notion.com/p/6f765df5069a4053a5729a892f8681be)
- [5.3.1 CaaS Delivery Process](https://app.notion.com/p/69835effa02a420ab72462836d108218)

## What the documentation does NOT answer

Say so plainly rather than improvising — these are real gaps, verified absent:

- **No field list for the ticket form**, and no list of which form types drive
  which pipeline. The routing rule is documented; the mapping is not.
- **No customer-visible ticket status/stage vocabulary.**
- **No access-control model.** The HubSpot guide says a customer can see "any
  other tickets from their company", but nothing documents how that company
  link is established, who at a customer can see what, or what happens when
  someone signs up with a wrong-domain address.
- **No SLA published to customers.** Internal response frames (6h / 6h / 12h)
  exist only inside the Maintenance process and are not a customer commitment.
- **No password reset, account administration or offboarding procedure.**

If a customer asks about any of the above, the honest answer is that it is not
documented — escalate to the Queue Manager or Process Owner for that pipeline
rather than inventing a rule.
