# Security

## Reporting Vulnerabilities

Report issues via [GitHub Issues](https://github.com/digitaloconnor/ai-news-briefing/issues) or [LinkedIn](https://linkedin.com/in/digitaloconnor).

Do not include API keys, tokens, or credential values in any report.

## Data Handled

**What this workflow touches:**

| Data | Source | Sent Externally? |
|------|--------|-----------------|
| Article titles, descriptions, URLs | NewsAPI, GNews | Sent to OpenAI (GPT-4o-mini) for summarisation |
| Article metadata (author, source, date) | NewsAPI, GNews | Sent to OpenAI |
| AI-generated briefing text | OpenAI | Posted to Slack channel |

**What this workflow does NOT touch:**
- No personal data
- No user data
- No internal system data
- No email content

**OpenAI data handling:** Article titles and descriptions are sent to the OpenAI API. Organisations must verify their OpenAI API agreement meets applicable data protection requirements (GDPR etc.) before using this workflow in production.

## Security Controls

- API keys stored in n8n credential store (HTTP Header Auth) — not hardcoded in workflow JSON
- `.env` file excluded from version control via `.gitignore`
- No inbound webhooks — outbound-only traffic
- OAuth2 tokens for Slack, Outlook, and LinkedIn managed by n8n with automatic refresh
- No sensitive data logged to n8n execution history (article content is not personally identifiable)

## Network Exposure

**Inbound:** None. Triggered by internal cron schedule only.

**Outbound:**
- `newsapi.org` — article fetch
- `gnews.io` — article fetch
- `api.openai.com` — GPT-4o-mini summarisation
- `slack.com` — briefing delivery
- `graph.microsoft.com` — Outlook (when configured)
- `api.linkedin.com` — LinkedIn (when configured)
