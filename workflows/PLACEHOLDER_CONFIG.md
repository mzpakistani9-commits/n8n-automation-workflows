# Placeholder configuration notes

All workflows import as-is. Before triggering, connect real credentials in the n8n editor:

| Node | Replace with |
|---|---|
| `airtable` nodes | Your Airtable API key in n8n credentials + real Table ID |
| `googleSheets` node | A Google service account / OAuth with access to the `Contacts` sheet |
| `emailSend` nodes | Your SMTP credentials (or Connect + Gmail/Outlook) |
| `slack` nodes | A Slack bot token with chat:write + a real channel |
| `openAi` nodes | `OPENAI_API_KEY` env var or n8n credential |
| `$env.HUBSPOT_TOKEN` | HubSpot private app token in n8n env vars |

See README → Import into your n8n for the quick start.
