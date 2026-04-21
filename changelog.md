# Changelog

## v1.0 — 2026-04-21

- Production release
- Dual-source fetch (NewsAPI + GNews) with parallel execution
- Deduplication by URL, 24-hour recency filter
- Engagement scoring by keyword relevance and source reputation
- GPT-4o-mini briefing: executive summary, top trending, latest, key insights, hashtags
- Slack output operational
- GNews API key moved from inline query parameter to HTTP Header Auth credential (security fix)
- `Create a post` node renamed to `LinkedIn: Publish Post`
- 5 section-level sticky notes added
- Outlook and LinkedIn nodes present but flagged as incomplete

## v0.1 — 2026-03-28

- Initial build
- Single-pass fetch and summarize
- GNews API key hardcoded inline (security issue — fixed in v1.0)
- No sticky notes
- Slack output only functional channel
