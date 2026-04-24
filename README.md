# 📧 MailClaw

> Turn your Gmail inbox into an automation hub — create rules, view AI-analyzed emails, and connect apps like Slack, Notion, and Calendar.

## Install

GitHub: https://github.com/tourmind-com/mailclaw

Install with the Skills CLI:

```bash
npx skills add tourmind-com/mailclaw
```

For ClawHub, use the same GitHub repository URL when publishing or installing MailClaw.

## Skill Files

| File | Purpose |
|------|---------|
| [SKILL.md](./SKILL.md) | Core skill definition — intents, API reference, output formats |
| [heartbeat.md](./heartbeat.md) | Daily digest routine — fetch pre-analyzed emails and report |

## Quick Start

1. **Get your API key** at https://aauth-170125614655.asia-northeast1.run.app/dashboard
2. The skill will save it to `config.json` along with app authorization status
3. Connect Gmail via the skill (it will guide you through OAuth — the server automatically sets up Pub/Sub for real-time email processing)
4. Create your first rule — e.g. "when I get a meeting invite, create a calendar event"

## Default Rules

MailClaw ships with these recommended rules:

| Rule | Condition | App | Action |
|------|-----------|-----|--------|
| Meeting → Calendar | Emails containing meeting invites, schedules, or calendar requests | `googlecalendar` | `GOOGLECALENDAR_CREATE_EVENT` |
| Task → Notion | Emails assigning tasks, requesting deliverables, or with action items | `notion` | `NOTION_CREATE_NOTION_PAGE` |
| Client emails → Notify | Emails from important contacts or clients | `slack` | `SLACK_SENDS_A_MESSAGE_TO_A_SLACK_CHANNEL` |

Example prompts:
- "When I receive a meeting invite, create a Google Calendar event"
- "When I get an email with action items, create a Notion page"

## Heartbeat 💓

Email analysis happens server-side via Pub/Sub — no periodic polling needed. The heartbeat tasks below are for daily digest summaries only. OpenClaw reads the `<!-- openclaw:heartbeat -->` block below and registers the tasks automatically.

<!-- openclaw:heartbeat
tasks:
  - name: daily-digest-morning
    interval: 24h
    prompt: "Generate a morning email digest. Read {baseDir}/heartbeat.md and follow every step exactly. Fetch unprocessed emails via GET /emails?unprocessed_only=true. Use the output formats defined in heartbeat.md — 📌 for matched emails, ☀️ for unmatched. Generate a processing page link via POST /daily-token/generate. Output the digest even if no rules matched."

  - name: daily-digest-evening
    interval: 24h
    prompt: "Generate an evening email digest. Read {baseDir}/heartbeat.md and follow every step exactly. Fetch unprocessed emails via GET /emails?unprocessed_only=true. Use the output formats defined in heartbeat.md — 📌 for matched emails, ☀️ for unmatched. Generate a processing page link via POST /daily-token/generate. Output the digest even if no rules matched."
-->

### Registered Tasks

| Task | Interval | Time | What it does |
|------|----------|------|-------------|
| `daily-digest-morning` | 24h | ~09:00 | Morning summary of overnight emails |
| `daily-digest-evening` | 24h | ~17:00 | End-of-day summary of today's emails |

### Server-Side Email Processing

When Gmail authorization completes, the server automatically registers a Google Pub/Sub subscription. New emails are analyzed server-side in real-time — intent classification, summarization, and rule matching all happen automatically without client involvement.

- **Matched emails** (📌) — hit a rule, presented individually with suggested actions for user confirmation
- **Unmatched emails** (☀️) — no rule hit, combined into a digest block with a processing page link + verification code

See [SKILL.md](./SKILL.md) and [heartbeat.md](./heartbeat.md) for the exact output formats.

## How It Works

```
┌─────────────┐     ┌──────────────┐     ┌──────────────┐
│  Server-Side │     │   Trigger    │     │   Output     │
│  (automatic) │     │              │     │              │
│ • Pub/Sub    │     │ • User query │     │ • Notify     │
│ • Analyze    │────▶│ • Digest     │────▶│ • 📌 / ☀️    │
│ • Match rules│     │ • Confirm    │     │ • Execute    │
└─────────────┘     └──────────────┘     └──────────────┘
```

### Category Tags

| Category | When | Frontend Tag |
|----------|------|-------------|
| **Realtime** 📌 | Email matched a rule or has suggested actions | Green tag, action buttons |
| **Brief** ☀️ | Email matched no rules | Purple tag, view only |

### Supported Apps

| App | Status | Common Actions |
|-----|--------|---------------|
| Gmail | Core | Send, reply, fetch, label |
| Google Calendar | Integration | Create event, quick add, find events |
| Notion | Integration | Create page, insert database row |
| Slack | Integration | Send message to channel |
| Linear | Integration | Create issue |
| HubSpot | Integration | Create contact, deal, note |

