# Make.com equivalents

Same business logic as the n8n workflows in `../workflows`, built for Make.com (Visual scenario builder).

## How to use

1. Log in to Make.com → **Scenarios** → **Create a new scenario**.
2. Add the modules below in order and connect them.
3. Every Make scenario = **Trigger → AI module → Router/If → App modules**, mirrored from the n8n graph.

## [1] Lead Intake & Qualification

| Make module | n8n equivalent | Setting |
|---|---|---|
| **Webhooks** → Custom webhook | Webhook (Lead form) | `lead-intake` |
| **Tools** → Data store / Text parser | Code: Normalize Lead | trim + lowercase |
| **OpenAI** → Create a Chat Completion | AI: Qualify Lead | gpt-4o-mini, `{score, tier, reason}` JSON |
| **Flow control** → Router | IF score >= 80 | `score >= 80` red → sales-ready path |
| **Airtable** → Add a record | CRM: Upsert Lead | table `Leads`, tier field |
| **Email** → Send an email | Email: Alert Owner | subject per tier path |

## [2] Support Ticket Triage

| Make module | n8n equivalent | Setting |
|---|---|---|
| **Webhooks** → Custom webhook | Webhook (Ticket created) | `ticket` |
| **OpenAI** → Create a Chat Completion | AI: Classify Ticket | `{category, sentiment, urgency, answerable}` |
| **Flow control** → Router | IF answerable = true | `answerable == true` |
| **HTTP** → Make a request (GET) | RAG Agent | `http://localhost:8000/ask?question=...` |
| **Flow control** → Router (inside) | Code: Post-process RAG | `escalated == false → email`, else Slack |
| **Slack** → Send a message | Slack: Escalate to Human | `#support` channel |

## [3] Daily Ops Report

| Make module | n8n equivalent | Setting |
|---|---|---|
| **Scheduler** → Every day | Schedule (Daily 09:00) | 09:00 PKT |
| **Data stores** / aggregator | Code: Pull Metrics | aggregate metrics |
| **OpenAI** → Create a Chat Completion | AI: Summarize Report | 3-4 sentence exec summary |
| **Email** → Send an email | Email: Ops Team | ops@example.com |
| **Slack** → Send a message | Slack: #ops-report | same body |

## [4] CRM & Notifications Sync

| Make module | n8n equivalent | Setting |
|---|---|---|
| **Webhooks** → Custom webhook | Webhook (CRM event) | `crm-event` |
| **Flow control** → Router | IF event = created | `event == created` |
| **Google Sheets** → Add a row | Google Sheets: Add Row | `Contacts` sheet |
| **HTTP** → Make a request (POST) | HubSpot: Upsert Contact | Bearer `{{HUBSPOT_TOKEN}}` |
| **Slack** → Send a message | Slack: #sales | notify team |

## [5] Social Media Lead Capture

| Make module | n8n equivalent | Setting |
|---|---|---|
| **Instagram** / **Meta** trigger | Webhook (comment/DM) | listen to comments/DMs |
| **Text parser** | Code: Extract Lead | intent regex |
| **OpenAI** → Create a Chat Completion | AI: Buying Intent | `{isLead: bool}` |
| **Flow control** → Router | IF isLead = true | `isLead == true` |
| **HTTP** → Make a request (POST) | Forward to Lead Pipeline | `http://localhost:5678/webhook/lead-intake` |

## [6] Email Follow-up Sequence

| Make module | n8n equivalent | Setting |
|---|---|---|
| **Scheduler** → Every X hours | Schedule (Every 3h) | every 3 hours |
| **Airtable** → Search records | Airtable: Due nurture leads | `stage = nurture` |
| **Flow control** → Router | IF still nurture | filter out converted |
| **Email** → Send an email | Email: Follow-up | template + merge fields |
| **Airtable** → Update a record | Airtable: bump lastTouch | set `lastTouch = today` |

> The Router modules in Make use **"Automatically choose a route"** with an `if` expression — e.g. `{{3.score}} >= 80`. `3` is the module index of the OpenAI result in your scenario.