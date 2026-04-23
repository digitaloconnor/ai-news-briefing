# 📰 AI News Briefing

> **Daily AI news aggregation — two sources, ranked by relevance, summarised by GPT-4o-mini, delivered to Slack**

[![Status](https://img.shields.io/badge/Status-Active-green)]()
[![Category](https://img.shields.io/badge/Category-Comms-blue)]()
[![n8n](https://img.shields.io/badge/n8n-v1.121.3-orange)]()
[![License](https://img.shields.io/badge/License-MIT-lightgrey)]()

---

## 📋 Table of Contents
- [Overview](#-overview)
- [Workflow Architecture](#-workflow-architecture)
- [Node Table](#-node-table)
- [Credentials Required](#-credentials-required)
- [Quick Start](#-quick-start)
- [Customisation](#-customisation)
- [Error Handling](#-error-handling)
- [Technologies Used](#️-technologies-used)
- [Known Limitations](#-known-limitations)
- [Changelog](#-changelog)
- [Contact](#-contact)

---

## 🎯 Overview

This workflow runs every morning at 9 AM. It pulls up to 100 AI-related articles from **NewsAPI** and **GNews** in parallel, deduplicates by URL, scores each article by keyword relevance and source reputation, and produces two ranked lists — top 10 by engagement and top 10 by recency. GPT-4o-mini turns these into a structured briefing and posts it to Slack.

### Why This Matters
- **Signal over noise** — scoring filters out low-value articles before they reach the AI summariser
- **Dual-source coverage** — NewsAPI and GNews have different indexing, so combining them catches more relevant stories
- **Alert fatigue prevention** — a single structured daily briefing beats 50 individual notifications
- **Consistent format** — executive summary, trending stories, latest developments, key insights, topic hashtags every day

### Key Features
- ✅ Parallel fetch from two news APIs
- ✅ URL deduplication across sources
- ✅ Keyword + source reputation scoring
- ✅ GPT-4o-mini structured briefing
- ✅ Slack delivery with markdown formatting

---

## 🏗️ Workflow Architecture

```
Daily at 9 AM
      │
      ├──────────────────────────┐
      ▼                          ▼
NewsAPI: AI Articles       GNewsAPI: AI Tech
(up to 50 articles)        (up to 50 articles)
      │                          │
      └────────────┬─────────────┘
                   ▼
        Merge All News Sources
                   │
                   ▼
        Filter & Normalize Articles
        (dedupe by URL, drop >24h old,
         score by keywords + source)
                   │
        ┌──────────┴──────────┐
        ▼                     ▼
 Top 10 Popular          Top 10 Newest
 (by engagement          (by publish date)
  score)
        │                     │
        └──────────┬───────────┘
                   ▼
         Merge Popular + Newest
                   │
                   ▼
         Prepare for AI Agent
         (serialize to JSON string)
                   │
                   ▼
    AI Agent: Summarize News ◄── OpenAI GPT-4o-mini
                   │
                   ▼
          Slack: Post Summary
```

---

## 📊 Node Table

| Node | Type | Purpose |
|------|------|---------|
| Daily at 9 AM | Schedule Trigger | Fires at 09:00 daily via cron |
| NewsAPI: AI Articles | HTTP Request | Fetches up to 50 articles from newsapi.org |
| GNewsAPI: AI Tech | HTTP Request | Fetches up to 50 articles from gnews.io |
| Merge All News Sources | Merge | Combines both source responses |
| Filter & Normalize Articles | Code | Deduplicates by URL, drops stale articles, calculates engagement score |
| Top 10 Popular Articles | Code | Sorts by engagement score, returns top 10 |
| Top 10 Newest Articles | Code | Sorts by publish date descending, returns top 10 |
| Merge Popular + Newest | Merge | Combines ranked lists for AI processing |
| Prepare for AI Agent | Set | Serialises article array to JSON string for prompt |
| AI Agent: Summarize News | AI Agent | Generates structured briefing |
| OpenAI Chat Model | LM Chat (OpenAI) | GPT-4o-mini powering the agent |
| Slack: Post Summary | Slack | Posts briefing as block message |

---

## 🔑 Credentials Required

| Credential | Type | Used By |
|------------|------|---------|
| NewsAPI Key | HTTP Header Auth | NewsAPI: AI Articles |
| GNews API Key | HTTP Header Auth | GNewsAPI: AI Tech |
| OpenAI API Key | n8n OpenAI credential | OpenAI Chat Model |
| Slack OAuth2 | n8n Slack credential | Slack: Post Summary |

---

## 🚀 Quick Start

### Prerequisites
- n8n instance (self-hosted or cloud)
- [NewsAPI](https://newsapi.org/) API key (free tier: 100 requests/day)
- [GNews](https://gnews.io/) API key (free tier: 100 requests/day)
- OpenAI API key
- Slack workspace with bot permissions

### Setup Steps

**1. Import the workflow**
```bash
# Via n8n CLI
n8n import:workflow --input=ai-news-briefing.json

# Or import via n8n UI: Settings → Import from File
```

**2. Configure credentials**

In n8n → Credentials, create:
- **HTTP Header Auth** for NewsAPI (`Authorization: Bearer YOUR_KEY`)
- **HTTP Header Auth** for GNews (`X-API-Key: YOUR_KEY`)
- **OpenAI** credential with your API key
- **Slack OAuth2** credential

**3. Set your Slack channel**

In `Slack: Post Summary`, update `channelId` to your target channel.

**4. Copy environment template**
```bash
cp .env.example .env
# Fill in your keys
```

**5. Test before activating**

Run the workflow manually once. Verify the Slack message format looks correct before enabling the daily schedule.

---

## ⚙️ Customisation

**Search terms** — Edit the `q` parameter in both HTTP Request nodes:
```
artificial intelligence OR AI models OR machine learning OR GPT OR Claude OR MCP
```

**Schedule** — Modify the cron expression in `Daily at 9 AM`:
- Weekdays only at 7 AM: `0 7 * * 1-5`
- Twice daily: `0 8,17 * * *`

**Engagement scoring** — In `Filter & Normalize Articles`, adjust:
- `highEngagementKeywords` — terms that boost score
- `reputableSources` — sources that receive a reputation bonus

**Article count** — Change `pageSize` (NewsAPI) and `max` (GNews) from 50 to adjust volume.

---

## 🛡️ Error Handling

- Both HTTP Request nodes use `onError: continueRegularOutput` — one failed source won't stop the run
- Slack node also continues on error
- No failure notification is currently configured — add an Error Trigger node connected to a Slack alert if uptime matters

---

## 🛠️ Technologies Used

| Tool | Version | Purpose |
|------|---------|---------|
| n8n | v1.121.3 | Workflow orchestration |
| NewsAPI | v2 | Primary AI news source |
| GNews | v4 | Secondary AI news source |
| OpenAI GPT-4o-mini | Latest | Article summarisation and briefing generation |
| Slack | API v2 | Briefing delivery |

### Why These Tools?

**NewsAPI** — Broad coverage, reliable API, free tier sufficient for daily use. Best-known aggregator for English-language tech news.

**GNews** — Covers sources that NewsAPI misses. Different indexing = better combined recall.

**GPT-4o-mini** — Fast and cheap for summarisation tasks where GPT-4 quality isn't required. Handles structured output well. Swap to GPT-4o if summary quality needs to improve.

**Slack** — Where the team already lives. No new tool adoption required.

---

## ⚠️ Known Limitations

- **Outlook and LinkedIn nodes present but incomplete** — not functional, not wired to content
- **Engagement scoring is heuristic** — keyword matching is a proxy for actual engagement data
- **console.log logging only** — results aren't queryable; add a Sheets/Notion logging step for trend analysis
- **Free API rate limits** — NewsAPI and GNews free tiers limit to ~100 requests/day each

---

## 📝 Changelog

See [changelog.md](changelog.md) for full version history.

**Latest:** v1.0 — 2026-04-21 — Production release with dual-source fetch, engagement scoring, and GPT-4o-mini summarisation.

---

## 📫 Contact

**Tony O'Connor**
- GitHub: [@digitaloconnor](https://github.com/digitaloconnor)
- n8n Instance: [automation.fioslabs.org](https://automation.fioslabs.org)

---

## 🔖 Tags

`#n8n` `#automation` `#ai-news` `#newsapi` `#gnews` `#openai` `#gpt4` `#slack` `#daily-briefing` `#news-aggregation` `#comms-automation`

---

**⚡ Status**: Active | **🗂️ Category**: Comms | **📅 Last Updated**: 2026-04-21
