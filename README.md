# 📧 MailClaw

> Turn your Gmail inbox into an automation hub — rules that react to emails, AI-powered analysis, and actions across connected apps.

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
| [heartbeat.md](./heartbeat.md) | Periodic email check routine — fetch, analyze, match, store, report |
| [references/actions.md](./references/actions.md) | Action index — load per-app action files on demand |
| [references/actions/*.md](./references/actions/) | Per-app action names and exact parameter formats |

## Quick Start

1. **Get your API key** at https://aauth-170125614655.asia-northeast1.run.app/dashboard
2. Save it to `api_key.txt` in this skill directory
3. Connect Gmail via the skill (it will guide you through OAuth)
4. Create your first rule — e.g. "when I get a meeting invite, create a calendar event"
5. Set up the Heartbeat to auto-check every 5 minutes

## Default Rules

MailClaw ships with these recommended rules. You can create them by telling the agent:

| Rule | Condition | App | Action |
|------|-----------|-----|--------|
| Meeting → Calendar | Emails containing meeting invites, schedules, or calendar requests | `googlecalendar` | `GOOGLECALENDAR_CREATE_EVENT` |
| Task → Notion | Emails assigning tasks, requesting deliverables, or with action items | `notion` | `NOTION_CREATE_NOTION_PAGE` |
| Client emails → Notify | Emails from important contacts or clients | `slack` | `SLACK_SENDS_A_MESSAGE_TO_A_SLACK_CHANNEL` |

To create a rule, tell the agent something like:
- "When I receive a meeting invite, create a Google Calendar event"
- "When I get an email with action items, create a Notion page"

The agent will confirm the rule before saving.

## Set Up Your Heartbeat 💓

Add MailClaw to your OpenClaw heartbeat to automatically check for new emails and process them through your rules.

### Step 1: Add to your HEARTBEAT.md

Add this task to your OpenClaw workspace `HEARTBEAT.md`:

```yaml
tasks:
  - name: email-check
    interval: 5m
    prompt: "Read the file at ~/.openclaw/skills/mailclaw/heartbeat.md and follow every step exactly. Do not skip any step. Do not deviate from the output format defined in that file."

  - name: daily-digest-morning
    interval: 24h
    prompt: "Generate a morning email digest. Fetch all emails from the last 24 hours via GET /emails?limit=50. Summarize them in the ☀️ digest format. Generate a processing page link via POST /daily-token/generate. Output the digest even if no rules matched — this is a summary for the user to review their inbox."

  - name: daily-digest-evening
    interval: 24h
    prompt: "Generate an evening email digest. Fetch all emails received today via GET /emails?limit=50. Summarize them in the ☀️ digest format. Generate a processing page link via POST /daily-token/generate. Output the digest even if no rules matched."
```

### Step 2: Configure heartbeat timing

In your `openclaw.json`, set the base heartbeat interval:

```json
{
  "agents": {
    "defaults": {
      "heartbeat": {
        "every": "1m",
        "target": "last"
      }
    }
  }
}
```

The base interval (1m) is how often the heartbeat runner checks for due tasks. Individual tasks have their own intervals, so this keeps the 5-minute email task close to schedule without running it every minute:
- `email-check`: every 5 minutes — fetches unprocessed emails, matches rules, stores analysis
- `daily-digest-morning`: every 24 hours — scheduled for ~09:00
- `daily-digest-evening`: every 24 hours — scheduled for ~17:00

### Step 3: What happens on each heartbeat

The `email-check` task runs the full processing cycle defined in `heartbeat.md`:

```
Fetch unprocessed emails
  → Load rules
  → Analyze each email (intent, summary)
  → Match against rules
  → Store analysis via mark-processed
  → Report to user
```

**Matched emails** get individual notifications:
```
📌 [Client email] David Kim sent an email

Q3 proposal final revisions: budget $48k, delivery moved up to 7/18

Suggested action: Create calendar event
[✓ Create] [✗ Skip] [→ View details]
```

**Unmatched emails** get batched into a digest:
```
☀️ Email Digest · Apr 14

3 emails pending:
• Sarah Lee: Asking about next week's schedule
• GitHub: PR #142 awaiting review
• Product Hunt: Daily featured picks

[→ Open processing page] (link valid for 24h)
```

### Daily Digest Schedule

The daily digest tasks push a summary of all emails at key times:

| Time | Task | What it does |
|------|------|-------------|
| ~09:00 | `daily-digest-morning` | Morning summary of overnight emails |
| ~17:00 | `daily-digest-evening` | End-of-day summary of today's emails |

The digest uses the ☀️ format and always includes a processing page link (valid for 24h) so you can handle emails from the web UI.

## How It Works

```
┌─────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Trigger    │────▶│  Fetch Data  │────▶│   Analyze    │────▶│   Output     │
│              │     │              │     │              │     │              │
│ • Heartbeat  │     │ • Emails     │     │ • Match rules│     │ • Store      │
│ • User query │     │ • Rules      │     │ • Classify   │     │ • Notify     │
│ • Digest     │     │              │     │ • Summarize  │     │ • 📌 / ☀️    │
└─────────────┘     └──────────────┘     └──────────────┘     └──────────────┘
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

## Additional Instructions

- If nothing needs attention, reply `HEARTBEAT_OK` — do not generate empty reports
- Do not repeat emails that were already processed in a previous cycle
- Keep all notifications concise and actionable
- When executing actions, use exact tool names from `references/actions/*.md`
- Parameter names are case-sensitive — always check the reference files before calling
