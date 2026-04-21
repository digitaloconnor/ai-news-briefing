# AI News Briefing

A daily n8n automation that fetches AI news from two sources, ranks and deduplicates articles, generates a structured briefing using GPT-4o-mini, and distributes it to Slack.

---

## What It Does

Runs at 9 AM every day. Pulls up to 100 AI-related articles from NewsAPI and GNews in parallel, deduplicates by URL, scores each article by keyword relevance and source reputation, and produces two ranked lists: top 10 by engagement and top 10 by recency. GPT-4o-mini turns these into a structured briefing with an executive summary, trending stories, latest developments, key insights, and topic hashtags. The final briefing is posted to a Slack channel.

Two additional output channels (Outlook email, LinkedIn post) are present in the workflow but incomplete — see [Known Limitations](#known-limitations).

---

## Workflow Diagram

```
Daily at 9 AM
      │
      ├──────────────────────┐
      ▼                      ▼
NewsAPI: AI Articles    GNewsAPI: AI Tech
      │                      │
      └──────────┬───────────┘
                 ▼
      Merge All News Sources
                 │
                 ▼
      Filter & Normalize Articles
      (dedupe, date filter, score)
                 │
      ┌──────────┴──────────┐
      ▼                     ▼
Top 10 Popular        Top 10 Newest
      │                     │
      └──────────┬───────────┘
                 ▼
       Merge Popular + Newest
                 │
                 ▼
       Prepare for AI Agent
                 │
                 ▼
      AI Agent: Summarize News ←── OpenAI Chat Model (GPT-4o-mini)
                 │
                 ▼
      LinkedIn: Publish Post  [⚠️ incomplete]
                 │
                 ▼
      Slack: Post Summary
      Outlook: Send Summary   [⚠️ incomplete]
```

---

## Node Table

| Node | Type | Purpose |
|------|------|---------|
| Daily at 9 AM | Schedule Trigger | Fires workflow at 09:00 every day |
| NewsAPI: AI Articles | HTTP Request | Fetches up to 50 articles from newsapi.org |
| GNewsAPI: AI Tech | HTTP Request | Fetches up to 50 articles from gnews.io |
| Merge All News Sources | Merge | Combines both source responses |
| Filter & Normalize Articles | Code | Deduplicates by URL, drops articles >24h old, calculates engagement score |
| Top 10 Popular Articles | Code | Sorts by engagement score, returns top 10 |
| Top 10 Newest Articles | Code | Sorts by publication date, returns top 10 |
| Merge Popular + Newest | Merge | Combines ranked lists for AI processing |
| Prepare for AI Agent | Set | Serialises article array to JSON string for prompt injection |
| AI Agent: Summarize News | AI Agent | Generates structured briefing from articles |
| OpenAI Chat Model | LM Chat (OpenAI) | GPT-4o-mini sub-node powering the agent |
| Slack: Post Summary | Slack | Posts briefing as block message to channel |
| Outlook: Send Summary | Microsoft Outlook | ⚠️ Incomplete — parameters not configured |
| LinkedIn: Publish Post | LinkedIn | ⚠️ Incomplete — no post content wired up |

---

## Credentials Required

| Credential | Type | Used By |
|------------|------|---------|
| NewsAPI Key | HTTP Header Auth (`Authorization: Bearer <key>`) | NewsAPI: AI Articles |
| GNews API Key | HTTP Header Auth (`X-API-Key: <key>`) | GNewsAPI: AI Tech |
| OpenAI API Key | n8n OpenAI credential | OpenAI Chat Model |
| Slack OAuth | n8n Slack credential | Slack: Post Summary |
| Microsoft Outlook OAuth2 | n8n Microsoft Outlook credential | Outlook: Send Summary |
| LinkedIn OAuth2 | n8n LinkedIn credential | LinkedIn: Publish Post |

---

## Setup

1. Copy `.env.example` to `.env` and fill in your API keys
2. In n8n, create credentials for NewsAPI, GNews, OpenAI, and Slack (see table above)
3. In `Slack: Post Summary`, set your target channel ID
4. Import `ai-news-briefing.json` into n8n (or deploy via `scripts/deploy.sh`)
5. Test manually before activating — run once and verify Slack output
6. Activate the workflow

---

## Customisation

**Search terms** — Edit the `q` parameter in both HTTP Request nodes. Current query covers: `artificial intelligence OR AI models OR machine learning OR GPT OR Claude OR MCP`.

**Briefing schedule** — Modify the cron expression in `Daily at 9 AM`. Current: `0 9 * * *` (9 AM daily). Change to `0 7 * * 1-5` for weekdays only at 7 AM.

**Engagement scoring** — Adjust `highEngagementKeywords` and `reputableSources` arrays in `Filter & Normalize Articles` to tune what gets ranked as high-value.

---

## Error Handling

Both HTTP Request nodes use `onError: continueRegularOutput` — if one news source fails, the other continues. Slack also continues on error. There is no error notification; if the briefing fails silently, add an Error Trigger node connected to a Slack alert.

---

## Known Limitations

- **Outlook: Send Summary** — node is present but has empty parameters. Not functional.
- **LinkedIn: Publish Post** — node is present but no post content is mapped. Not functional.
- **GNews API key** — the original workflow had the key hardcoded inline. This version uses HTTP Header Auth credentials. You must create the n8n credential before activating.
- **Engagement scoring** — the scoring algorithm is a heuristic proxy, not actual engagement data.

---

## Related Workflows

<!-- add links -->
