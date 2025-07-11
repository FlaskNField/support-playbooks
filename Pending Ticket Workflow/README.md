
# Pending Ticket Workflow

This workflow helps make sure tickets in a `pending` state are moving forward by automatically following up with the customer. If there's no response after a few reminders, the system will close the ticket — but not before giving the agent a heads-up.

## Overview

| Time in Pending     | Action                      | Customer Message | Agent Notification     |
|---------------------|-----------------------------|------------------|------------------------|
| 2 business days     | First reminder              | ✅               | —                      |
| 5 business days     | Final reminder + warning    | ✅               | —                      |
| 7 business days     | Auto-close                  | ✅               | ✅ (Slack + internal note) |

---

## Automation: 2-Day Pending Reminder

Sends a friendly nudge to the customer 2 business days after a ticket goes into `Pending`.

### Trigger Conditions

All of the following must be true:

- `Ticket: Status` is `Pending`  
- `Ticket: Brand` is `{{YOUR COMPANY}}`  
- `Ticket: Hours since pending (business)` is `16`  
- `Ticket: Tags` does **not** contain `missed_reminder`

### Actions

- Sends an email to the customer using the following template:

```text
Subject: Is this still something you need help with? [Sentry Support #{{ticket.id}}]

Hey there {{ticket.requester.first_name}},

Hope you're doing great! Just a quick nudge about your ticket – we're really looking forward to hearing back from you.

Oh, and here's a quick recap of your request:

{{ticket.comments_formatted}}

Cheers,  
The {{YOUR COMPANY}} Support Team
```

- (Optional) Adds tag: `first_reminder_sent`

---

## Automation: 5-Day Pending Reminder + Closure Warning

Fires 5 business days after a ticket goes into `Pending`, assuming the customer hasn’t responded. Gives them a heads-up that we’ll be closing the ticket soon and notifies the assigned agent.

### Trigger Conditions

All of the following must be true:

- `Ticket: Status` is `Pending`  
- `Ticket: Brand` is `{{YOUR COMPANY}}`  
- `Ticket: Hours since pending (business)` is `40`  
- `Ticket: Tags` does **not** contain `closure_warning_sent`

### Actions

- Sends the following email to the customer:

```text
Subject: Is this still something you need help with? [Sentry Support #{{ticket.id}}]

Hey {{ticket.requester.first_name}},

Just a quick reminder — your ticket is still open and pending your response.

If we don't hear back in the next 2 days, we'll go ahead and close it out.  
But no worries — if you need more time or want to reopen it later, just reply here.

Here's a recap of your original request:

{{ticket.comments_formatted}}

Thanks,  
The {{YOUR COMPANY}} Support Team
```

- Adds tag: `closure_warning_sent`

- Adds an internal note to the ticket:

```text
Internal Note:

Heads up — this ticket has been in Pending for 5 business days with no response from the customer.

Unless action is taken, this will be automatically closed in 2 days.
```

- Sends a Slack message to the assigned agent (via webhook/Zapier/Slack integration):

```text
:ticket: *Pending ticket auto-closure alert*

{{ticket.assignee.name}}, your ticket ({{ticket.id}}) has been in Pending for 5 business days with no customer response.

It will be auto-closed in 2 days unless you take action.  
Link: {{ticket.url}}
```

---

## Automation: 7-Day Auto-Close

After 7 business days of inactivity, this automation preps the ticket for auto-closure.

### Trigger Conditions

All of the following must be true:

- `Ticket: Status` is `Pending`  
- `Ticket: Brand` is `{{YOUR COMPANY}}`  
- `Ticket: Hours since pending (business)` is `56`  
- `Ticket: Tags` includes `closure_warning_sent`  
- `Ticket: Tags` does **not** contain `auto_closed_pending`

### Actions

```text
Subject: Closing your ticket for now — Sentry Support [#{{ticket.id}}]

Hi {{ticket.requester.first_name}},

We haven’t heard back from you, so we’re going to go ahead and close this ticket for now. No worries though — if you still need help, just reply to this email and we’ll pick things up right where we left off.

Here’s a recap of your original request, in case it’s helpful:

{{ticket.comments_formatted}}

Thanks again for reaching out,  
The {{YOUR COMPANY}} Support Team
```

---

## Reporting and Metrics

Tickets closed through this workflow will be tagged with `auto_closed_pending`. This makes it easy to:

- Track how many tickets are closing without full resolution
- Identify areas where follow-up or additional documentation could improve resolution rates
- Coach agents to drive down passive closures when possible

---

## Goals of This Workflow

- Keep the support queue clean and actionable  
- Nudge customers without requiring manual effort  
- Let agents retain control when needed  
- Provide visibility into closure reasons via tagging
