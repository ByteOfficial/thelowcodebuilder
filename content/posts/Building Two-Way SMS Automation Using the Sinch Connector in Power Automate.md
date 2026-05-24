---
title: "Building Two-Way SMS Automation Using the Sinch Connector in Power Automate"
date: 2026-03-16
lastmod: 2026-03-16
draft: false
slug: "two-way-sms-automation-sinch-power-automate"
categories: ["Power Automate"]
tags: ["Power Automate", "Sinch", "SMS", "Automation", "CRM", "tutorial", "advanced"]
description: "Create a two-way SMS experience: receive inbound texts, create CRM tickets, auto-reply, and track delivery receipts with message status."
ShowToc: true
TocOpen: true
cover:
  image: "/images/sms-automation-sinch-cover.png"
  alt: "Two-Way SMS Automation with Sinch and Power Automate"
  caption: "Build a complete two-way SMS automation pipeline"
  relative: false
---

# Building Two-Way SMS Automation Using the Sinch Connector in Power Automate

Two-way SMS is one of the fastest ways to reduce support friction: customers text you, your systems react instantly, and you can still track delivery outcomes for compliance and troubleshooting.

This post walks through a practical automation using recent **Sinch SMS Connector** updates:

- **Trigger:** *When receiving a delivery receipt*
- **Action:** *Get message status*
- **Trigger:** *Receive SMS* (for inbound messages)

We’ll build **two flows** (Power Automate supports **one trigger per flow**):
1. **Inbound SMS → Create CRM Ticket → Auto-reply**
2. **Delivery Receipt → Get Message Status → Update CRM Ticket**

---

## What you’ll build (high level)

**Customer sends SMS → Power Automate receives inbound message → Flow creates ticket in CRM → Flow sends automated reply → Delivery receipt updates ticket**

![Architecture overview: inbound SMS creates ticket + outbound reply; delivery receipt flow updates status](/images/2026-03-16/00-architecture-overview.png)

---

## Prerequisites

- A **Sinch account** with an SMS-enabled number (or short code/long code depending on region)
- Access to **Power Automate**
- A **CRM or ticketing system** (examples: Dynamics 365, Zendesk, Salesforce, HubSpot)  
  *If you don’t have one, you can use Dataverse, SharePoint list, or a simple Excel table as a “ticket store.”*

![Prerequisites checklist: Sinch number + Power Automate + CRM connection](/images/2026-03-16/01-prerequisites.png)

---

# Flow 1: Inbound SMS → Create Ticket → Auto-Reply

## Step 1 — Create a new cloud flow with the **Receive SMS** trigger

1. Go to **Power Automate → Create**
2. Choose **Automated cloud flow**
3. Name it: `Inbound SMS → Create Ticket → Reply`
4. Select trigger: **Sinch SMS — Receive SMS** (wording may appear as “When an SMS is received” / “Receive SMS”)

<!--
![Power Automate flow creation screen with Sinch "Receive SMS" trigger selected](images/02-create-flow-receive-sms.png)

> **Screenshot to capture:** The template picker with the Sinch trigger highlighted.
-->
---

## Step 2 — Configure trigger details (phone number, endpoint, keywords if applicable)

In the trigger, configure:
- The Sinch number (or service configuration) you want to receive messages on
- Any filtering options available (keywords, sender, etc.)

Typical fields you’ll see from inbound SMS:
- `From` (sender phone)
- `To` (your Sinch number)
- `Body` (message text)
- `MessageId` / `InboundMessageId` (varies by connector)

<!--
![Trigger configuration showing inbound fields like From/To/Body](images/03-trigger-config.png)


> **Screenshot to capture:** The trigger card expanded, showing the input configuration and dynamic content fields.
-->
---

## Step 3 — Create a ticket in your CRM

Add a CRM action such as:
- **Create record**
- **Create ticket**
- **Create case**
- **Create issue**

Map fields like:
- **Title/Subject:** `SMS from {From}`
- **Description:** `Body`
- **Contact/Phone:** `From`
- **Channel:** `SMS`
- **External Message ID:** store the inbound ID if available

<!--
![CRM "Create ticket/case" action with SMS fields mapped](images/04-crm-create-ticket.png)

> **Screenshot to capture:** The CRM action with mappings visible (Subject/Description/Phone).
-->
---

## Step 4 — Send an automated SMS reply via Sinch

Add the Sinch action: **Send SMS** (or equivalent).

Suggested reply template:

> “Thanks! We’ve received your message and created ticket #{TicketNumber}. Reply with more details anytime.”

If your CRM returns a ticket ID/number, insert it into the SMS body.

<!--
![Sinch "Send SMS" action configured to reply to the inbound sender](images/05-send-sms-reply.png)

> **Screenshot to capture:** The Send SMS card showing `To = From` and the message body using CRM ticket number.
-->
---

## Step 5 — Store correlation data (recommended)

To make delivery receipts easy to match back to tickets, store:
- The **outbound SMS Message ID** returned by Sinch (from the Send SMS step)
- The **CRM Ticket ID**

Where to store it:
- In the CRM ticket itself (custom fields), or
- In a small table (Dataverse/SharePoint) keyed by `OutboundMessageId`

<!--
![Example: writing Outbound Message ID back to the CRM ticket](images/06-store-message-id.png)

> **Screenshot to capture:** An “Update ticket” step writing `OutboundMessageId` into the ticket record.
-->
---

## Step 6 — Test Flow 1 end-to-end

1. Save the flow
2. Text your Sinch number from your phone
3. Confirm:
   - A ticket is created
   - You receive the auto-reply

<!-- ![Test run results: inbound trigger fired, CRM created ticket, SMS reply sent](images/07-test-flow-1-run.png)

> **Screenshot to capture:** The flow run screen showing green checkmarks on each step. -->

---

# Flow 2: Delivery Receipt → Get Message Status → Update Ticket

Delivery receipts (DLRs) let you answer: “Did the SMS actually deliver? When? Why not?”

## Step 1 — Create a new cloud flow with **When receiving a delivery receipt**

1. **Create → Automated cloud flow**
2. Name it: `Delivery Receipt → Update Ticket`
3. Trigger: **Sinch SMS — When receiving a delivery receipt**

<!-- ![New flow creation with Sinch "When receiving a delivery receipt" trigger](images/08-create-flow-delivery-receipt.png)

> **Screenshot to capture:** The trigger selection screen with the delivery receipt trigger highlighted. -->

---

## Step 2 — Parse the delivery receipt (Message ID, status, timestamp)

The delivery receipt typically includes:
- `MessageId` (links to your outbound message)
- `Status` (delivered/failed/queued/etc.)
- `StatusDetail` or error codes (when failed)
- Timestamps

<!-- ![Delivery receipt trigger outputs showing MessageId and Status](images/09-delivery-receipt-fields.png)

> **Screenshot to capture:** The trigger outputs/dynamic content panel showing receipt fields. -->

---

## Step 3 — Add action: **Get message status**

Add the Sinch action **Get message status** and pass the `MessageId` from the delivery receipt.

Why this helps:
- Delivery receipts can be brief; **Get message status** can provide richer or latest state details (depending on region/product configuration).

<!-- ![Sinch "Get message status" action using MessageId from the receipt](images/10-get-message-status.png)

> **Screenshot to capture:** The Get message status card with `MessageId` mapped. -->

---

## Step 4 — Find the related CRM ticket (using stored OutboundMessageId)

You now need to map `MessageId → Ticket`.

Common approach:
- Search CRM records where `OutboundMessageId == MessageId`
- Or look up in your correlation table (Dataverse/SharePoint)

Then update the ticket:
- **SMS Delivery Status** = delivered/failed
- **Delivered At** = timestamp
- **Failure Reason** (if any)

<!-- ![CRM lookup by OutboundMessageId, then update ticket with delivery status](images/11-update-ticket-status.png)

> **Screenshot to capture:** The “List records / Search” step + “Update record” step showing mappings. -->

---

## Step 5 — Add optional notifications (Teams/Email) for failures

For example, if status is `failed`, notify a support channel:

- Post a message in Teams:  
  “SMS delivery failed for ticket #123. Reason: {StatusDetail}”

<!-- ![Condition branch for failed status + Teams notification action](images/12-notify-on-failure.png)

> **Screenshot to capture:** A Condition control showing Delivered vs Failed branches. -->

---

## Step 6 — Test Flow 2

- Send a new inbound SMS (Flow 1 will reply)
- Wait for the delivery receipt to trigger Flow 2
- Confirm the CRM ticket is updated with delivery status

<!-- ![Flow 2 run history: delivery receipt received, status fetched, ticket updated](images/13-test-flow-2-run.png)

> **Screenshot to capture:** Flow run with receipt trigger and ticket update step succeeded. -->

---

# Example automation (as described)

1. **Customer sends SMS**
2. **Flow creates ticket in CRM**
3. **Sends automated reply**
4. **Delivery receipt updates ticket status** (delivered/failed)

<!-- ![Example timeline: inbound message → ticket created → reply sent → delivery confirmed](images/14-example-timeline.png) -->

---

# Tips, gotchas, and best practices

- **Use correlation IDs:** Always store the outbound `MessageId` on the ticket so delivery receipts can update the right record.
- **Handle duplicates:** Some systems can emit multiple receipt updates (e.g., queued → sent → delivered). Update the ticket with the latest status + timestamps.
- **Normalize phone numbers:** Store `From` in E.164 format if possible to avoid mismatches.
- **Rate limits and retries:** If your CRM throttles, add retry policy and/or queue updates in Dataverse.

<!-- ![Best practices checklist for correlation, dedupe, normalization](images/15-best-practices.png) -->

---

# Conclusion

With the **Receive SMS** trigger, **When receiving a delivery receipt** trigger, and **Get message status** action, you can build a robust two-way SMS support channel in Power Automate—fully automated, trackable, and easy to extend.