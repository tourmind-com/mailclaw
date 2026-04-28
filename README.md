# MailClaw

> Turn your Gmail inbox into an automation hub — create rules, view AI-analyzed emails, and connect apps like Slack, Notion, Calendar, Linear, and HubSpot.

## Install

```bash
npx skills add tourmind-com/mailclaw
```

## Quick Start

1. Open the skill — it will guide you through setup
2. Paste your API key from https://aauth-170125614655.asia-northeast1.run.app/dashboard
3. Connect Gmail via OAuth
4. Read `setup.md` and register the heartbeat tasks defined there
5. Create your first rule from a template

## What It Does

**Server-side** (automatic): Gmail Pub/Sub receives new emails → AI analyzes intent, summarizes, matches rules → stores results.

**Skill** (conversational): Fetches pre-analyzed results, presents them, and executes user-confirmed actions.

**Heartbeat** (scheduled): Polls for new emails every minute, plus daily digests at 09:00 and 17:00.

## Skill Files

| File | Purpose |
|------|---------|
| `SKILL.md` | Core skill — setup flow, intents, API reference, output formats |
| `heartbeat.md` | Daily digest routine — fetch analyzed emails and present |
| `setup.md` | OpenClaw config — registers heartbeat tasks after Gmail auth |

## Rule Templates

| Template | Condition | App | Action |
|----------|-----------|-----|--------|
| Client emails → Notion task | Emails from important contacts or clients | `notion` | `NOTION_CREATE_NOTION_PAGE` |
| Meeting invites → Calendar event | Emails containing meeting invites or calendar requests | `googlecalendar` | `GOOGLECALENDAR_CREATE_EVENT` |
| Feedback emails → Slack alert | Emails containing feedback, reviews, or complaints | `slack` | `SLACK_SENDS_A_MESSAGE_TO_A_SLACK_CHANNEL` |

## Supported Apps

Gmail, Google Calendar, Notion, Slack, Linear, HubSpot.

## Example Prompts

- "Check my email"
- "When I get a meeting invite, create a calendar event"
- "Show my rules"
- "Connect Slack"
- "Reply to David's email"

## Workflow

```
User installs skill
  → Setup: API key → Gmail OAuth
    → Read setup.md → register heartbeat tasks
      → Every 1 min: poll for new emails (heartbeat.md)
      → 09:00 / 17:00: push daily digest (heartbeat.md)
    → First rule from template
      → User opens skill to interact with emails (SKILL.md)
```
