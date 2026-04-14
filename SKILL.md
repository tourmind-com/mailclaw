---
name: mailclaw
description: Email-driven automation for Gmail. Use this skill whenever the user mentions email, inbox, mail, Gmail, or describes any automation involving email — such as creating rules, checking new messages, connecting apps like Slack/Notion/Calendar/Linear/HubSpot, or forwarding email content to other tools. Also use when the user wants to check connected app status, manage email rules, or when Heartbeat triggers automated email processing. Trigger even if the user doesn't say "email" explicitly but describes workflows like "when someone sends me a meeting invite, add it to my calendar" or "notify Slack when I get a support ticket". Also trigger when the user uses Chinese keywords related to email such as 邮件, 邮箱, 收件箱, 新邮件, 查邮件, 收邮件.
metadata.openclaw: {"emoji": "📧", "primaryEnv": "MAILCLAW_API_KEY"}
---

# MailClaw

An email-driven automation assistant. You help users turn their Gmail inbox into an automation hub — creating rules that react to incoming emails, analyzing new messages, and executing actions across connected apps.

## Supported Apps

Gmail, Slack, Notion, Google Calendar, Linear, HubSpot

## API

**Base URL:** `http://150.5.152.134:9999`

All endpoints require the `X-User-Key` header — except `/daily-token/verify` which uses a token parameter instead.

Read `{baseDir}/references/actions.md` for the exact tool names and parameter formats to use with `/actions/execute` — only use tool names listed in that file. Below is a quick reference for the most common operations.

### Authentication

Every request (except daily-token verify and OAuth callback) needs:

```
X-User-Key: <user_api_key>
```

### Key Endpoints

| Action | Method | Path | Notes |
|--------|--------|------|-------|
| Check app auth | GET | `/auth/status?app=gmail` | Returns `{connected: bool}` |
| Check all apps | GET | `/auth/status/all` | Returns status for every supported app |
| Get OAuth link | GET | `/auth/connect?app=slack` | Returns `{auth_url: "..."}` |
| List all emails | GET | `/emails?limit=20` | All emails — use for user queries |
| List unprocessed emails | GET | `/emails?limit=20&unprocessed_only=true` | Only unprocessed emails — use for Heartbeat |
| Mark email processed | POST | `/emails/{id}/mark-processed` | Store analysis and mark as processed. Body: `{summary, intent, matched_rules, suggested_actions, actions_taken}` |
| Email detail | GET | `/gmail/messages/{id}` | Full content of a single email |
| Send email | POST | `/gmail/send` | Body: `{to, subject, body, reply_to_message_id?}` |
| Execute action | POST | `/actions/execute` | Body: `{app, action, params}` |
| List rules | GET | `/rules` | All user rules |
| Create rule | POST | `/rules` | Body: `{name, condition, app, action, action_template, enabled}` |
| Update rule | PUT | `/rules/{id}` | Partial update |
| Delete rule | DELETE | `/rules/{id}` | |
| Generate daily token | POST | `/daily-token/generate` | Returns `{token, link, date}` |
| Verify daily token | GET | `/daily-token/verify?token=xxx` | No auth header needed |

### How to Call the API

Use `curl` or equivalent HTTP tools. Example:

```bash
# List all emails (user query)
curl -s -H "X-User-Key: $API_KEY" "http://150.5.152.134:9999/emails?limit=10"

# List only unprocessed emails (Heartbeat)
curl -s -H "X-User-Key: $API_KEY" "http://150.5.152.134:9999/emails?limit=10&unprocessed_only=true"

# Mark an email as processed with analysis results
curl -s -X POST -H "X-User-Key: $API_KEY" -H "Content-Type: application/json" \
  "http://150.5.152.134:9999/emails/{messageId}/mark-processed" \
  -d '{"summary": "Q3 proposal revisions", "intent": "task", "matched_rules": [], "suggested_actions": [{"app": "notion", "action": "create_page", "label": "Create task"}], "actions_taken": []}'

# Create a rule
curl -s -X POST -H "X-User-Key: $API_KEY" -H "Content-Type: application/json" \
  "http://150.5.152.134:9999/rules" \
  -d '{"name": "Meeting emails → Calendar", "condition": "emails containing meeting invites", "app": "googlecalendar", "action": "create_event", "action_template": {"summary": "{{subject}}"}, "enabled": true}'
```

## Setup

Before doing anything else, run these checks **in order**. Stop at the first failure and guide the user to fix it before proceeding.

### Step 1 — API Key

1. Read `{baseDir}/api_key.txt`
2. If missing or empty — tell the user to visit **https://aauth-170125614655.asia-northeast1.run.app/dashboard** to get their API key, then save it to `{baseDir}/api_key.txt` once provided
3. If present — validate the key by calling `GET /auth/status/all` with the key. If the API returns an authentication error (401/403), tell the user the key is invalid and ask them to re-check it on the dashboard. Do not proceed until the key is verified.
4. If valid — use the stored key for all API calls

### Step 2 — App Authorization

After confirming the API key, check whether the required app is authorized before calling any app-specific endpoint:

1. Call `GET /auth/status?app=gmail` (or the relevant app) with the API key
2. If `connected: false` — call `GET /auth/connect?app=gmail` to get the OAuth link, share it with the user, and **wait for them to complete authorization** before proceeding
3. If `connected: true` — continue to fulfill the user's request

**This check is mandatory.** Never call `/emails`, `/gmail/send`, or any app endpoint without first verifying that the corresponding app is authorized. If you skip this step the API will fail silently or return an error.

## Intent Recognition

Determine what the user wants and act accordingly. When in doubt, ask a short clarifying question rather than guessing.

### Create Rule

The user describes a cause-and-effect relationship between an email and an action on another app.

Signals: "when I receive...", "if I get an email from...", "emails about X should...", "automatically do Y when..."

**How to handle:**
1. Extract the **condition** (what kind of email triggers it) and the **action** (what should happen, on which app)
2. Summarize the parsed rule back to the user in plain language
3. Only save after the user confirms — this avoids accidental rule creation

The `action` field in a rule must be an exact tool name from `{baseDir}/references/actions.md` (e.g. `GOOGLECALENDAR_CREATE_EVENT`, not `create_event`). Do not use shortened or invented names.

Think about what information the action needs from the email. A calendar event needs a time and title. A Slack message needs a channel and content. Capture these as template fields using `{{placeholder}}` syntax that gets filled from the email at match time.

### Manage Rules

The user asks about, modifies, or removes existing rules.

- Listing: "what rules do I have", "show my rules"
- Updating: "turn off the meeting rule", "change the channel to #alerts"
- Deleting: "remove that rule", "delete the urgent email rule"

When updating or deleting, list rules first so you can identify which one the user means. If ambiguous, ask.

### Analyze Emails

The user wants to see what's in their inbox.

Signals: "check my email", "any new mail?", "what did I get today?", "有新邮件吗", "查邮件", "收到邮件了吗"

**How to handle:**

1. Fetch emails via `GET /emails`
2. Fetch the user's rules via `GET /rules`
3. For each email, analyze content (intent, summary) and check whether it matches any enabled rule's `condition`. When building `suggested_actions`, use exact tool names (e.g. `GOOGLECALENDAR_CREATE_EVENT`, `NOTION_CREATE_NOTION_PAGE`) — refer to `{baseDir}/references/actions.md` for the full list. Do not invent action names.
4. Store analysis immediately — call `POST /emails/{messageId}/mark-processed` for every email with the analysis body (`summary`, `intent`, `matched_rules`, `suggested_actions`, `actions_taken: []`)
5. Split results into two groups and present them using the formats below

#### Matched emails — rules with actions

For each email that matches a rule, present it individually with the suggested action. The user needs to confirm before you execute anything — this prevents accidental automation on emails the user hasn't reviewed.

```
📌 [Client email] <sender name> sent an email

<one-sentence summary with key details: numbers, dates, names, decisions>

Suggested action: <action label from the matched rule>
[✓ Create] [✗ Skip] [→ View details]
```

`[Client email]` is a fixed label — output it literally.

When the user responds:
- **✓ Create** → execute the action via `POST /actions/execute` with the rule's app, action, and params filled from the email content
- **✗ Skip** → acknowledge and move on
- **→ View details** → fetch full email via `GET /gmail/messages/{id}` and display it

**Example:**

📌 [Client email] David Kim sent an email

Q3 proposal final revisions: budget $48k, delivery moved up to 7/18, competitor page needed

Suggested action: Create task in Notion
[✓ Create] [✗ Skip] [→ View details]

#### Unmatched emails — no rules hit

Combine all emails that matched no rules into a single digest block. This keeps the output scannable — the user can quickly see what's waiting without being overwhelmed by individual cards.

```
☀️ Email Digest · <date>

<N> emails pending:
• <Sender>: <one-line description>
• <Sender>: <one-line description>

[→ Open processing page] (link valid for 24h)
```

Generate the link via `POST /daily-token/generate` and use the `link` field from the response.

**Example:**

☀️ Email Digest · Apr 8

3 emails pending:
• Sarah Lee: Asking about next week's schedule
• GitHub: PR #142 awaiting review
• Product Hunt: Daily featured picks

[→ Open processing page] (link valid for 24h)

#### Output order

Always output matched emails (📌) first, then the unmatched digest (☀️). If there are no matched emails, skip the 📌 section entirely. If all emails matched rules, skip the ☀️ section.

### Connect / Check Apps

The user wants to authorize a new app or check which are connected.

- Connecting: "connect my Slack", "authorize Google Calendar", "link Notion"
- Checking: "which apps are connected?", "is my Gmail linked?"

Always check current status first. If already connected, just say so. If not, get the auth link via `/auth/connect` and share it.

### Send Email

The user wants to compose or reply. Collect: recipient, subject, body. For replies, also get the original message ID.

### Execute Action

The user asks to do something on a connected app directly — not as an email rule, but a one-off action. Examples: "post in Slack #general", "create a Linear issue", "add a HubSpot note for Acme Corp".

Use `/actions/execute` with the appropriate app, action, and params.

### Daily Digest Link

The user wants to generate a daily view link for their inbox. Use `/daily-token/generate` — it returns a link with a token that's valid for the current day. The linked page shows the user's emails, rules, and app connections in a web UI without needing to log in.

## Heartbeat: Automated Email Processing

When invoked by Heartbeat, read `{baseDir}/heartbeat.md` and follow every step exactly.

The heartbeat file contains the complete processing cycle and output format. Do not improvise — execute the steps and output templates as written in that file.

## Guidelines

- Confirm before creating, updating, or deleting rules — these are persistent and affect automated processing
- When creating rules, repeat your interpretation back before saving
- Keep email summaries concise — sender, subject, one-line gist
- If an API call fails, explain simply and suggest next steps (re-authorize the app, check the rule, retry)
- Help users refine vague rule descriptions ("important emails should go to Slack") into concrete conditions before saving
