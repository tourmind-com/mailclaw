# MailClaw Heartbeat

This runs periodically to check for new emails and process them through your rules.

## Steps

Follow these steps **in order**. Do not skip any step. Do not deviate from the output format.

Before starting, output this line so the user knows the heartbeat file was loaded:

```
🫀 MailClaw Heartbeat running...
```

### Step 1: Fetch unprocessed emails

```
GET /emails?unprocessed_only=true
```

If the response contains **zero** emails → reply `HEARTBEAT_OK` and stop. Do not proceed.

### Step 2: Load rules

```
GET /rules
```

Keep the full list of enabled rules in memory for Step 3.

### Step 3: Analyze & Match

For **each** email from Step 1:

1. Read the email's `from`, `subject`, `body`, `timestamp`
2. Determine the **intent**: `meeting`, `task`, `notification`, `promotion`, `security`, or `other`
3. Write a one-line **summary** that captures the key information (numbers, dates, names, decisions)
4. Check the email against **every rule** from Step 2 — does the email content satisfy the rule's `condition`?
5. If matched, note the `rule_id`, `rule_name`, and build the **suggested action** with pre-filled `params` extracted from the email. Use the exact action names and param formats below — do not invent action names:

**Google Calendar** — action: `GOOGLECALENDAR_CREATE_EVENT`
Extract from email: meeting time, duration, title, location, attendees, timezone.
```json
{
  "app": "googlecalendar",
  "action": "GOOGLECALENDAR_CREATE_EVENT",
  "label": "Create calendar event",
  "params": {
    "summary": "<meeting title — this is the title field, NOT 'title'>",
    "start_datetime": "<REQUIRED, ISO 8601: 2026-04-18T14:00:00>",
    "event_duration_hour": 1,
    "event_duration_minutes": 0,
    "timezone": "<e.g. Asia/Shanghai or America/Los_Angeles>",
    "description": "<meeting details, agenda>",
    "location": "<location or meeting link>",
    "attendees": ["<email1>", "<email2>"],
    "create_meeting_room": true
  }
}
```

**Notion** — action: `NOTION_CREATE_NOTION_PAGE`
```json
{
  "app": "notion",
  "action": "NOTION_CREATE_NOTION_PAGE",
  "label": "Create Notion page",
  "params": {
    "title": "<REQUIRED, page title>",
    "parent_id": "<REQUIRED, database or page UUID>",
    "markdown": "<page content — use 'markdown', NOT 'content'>"
  }
}
```

**Slack** — action: `SLACK_SENDS_A_MESSAGE_TO_A_SLACK_CHANNEL`
```json
{
  "app": "slack",
  "action": "SLACK_SENDS_A_MESSAGE_TO_A_SLACK_CHANNEL",
  "label": "Send Slack message",
  "params": {
    "channel": "<REQUIRED, channel ID or name>",
    "text": "<message content, up to 4000 chars>"
  }
}
```

**Linear** — action: `LINEAR_CREATE_LINEAR_ISSUE`
```json
{
  "app": "linear",
  "action": "LINEAR_CREATE_LINEAR_ISSUE",
  "label": "Create Linear issue",
  "params": {
    "title": "<REQUIRED, issue title>",
    "team_id": "<REQUIRED, team UUID>",
    "description": "<issue description, supports markdown>",
    "priority": 3
  }
}
```

**HubSpot** — action: `HUBSPOT_CREATE_NOTE`
```json
{
  "app": "hubspot",
  "action": "HUBSPOT_CREATE_NOTE",
  "label": "Create HubSpot note",
  "params": {
    "hs_timestamp": "<REQUIRED, ISO-8601 UTC e.g. 2026-04-14T10:00:00Z>",
    "hs_note_body": "<note content — use 'hs_note_body', NOT 'body'>"
  }
}
```

**Gmail** — action: `GMAIL_SEND_EMAIL`
```json
{
  "app": "gmail",
  "action": "GMAIL_SEND_EMAIL",
  "label": "Send email",
  "params": {
    "recipient_email": "<use 'recipient_email', NOT 'to'>",
    "subject": "<email subject>",
    "body": "<email content>"
  }
}
```

**Gmail Reply** — action: `GMAIL_REPLY_TO_THREAD`
```json
{
  "app": "gmail",
  "action": "GMAIL_REPLY_TO_THREAD",
  "label": "Reply to thread",
  "params": {
    "thread_id": "<REQUIRED, hex string>",
    "message_body": "<reply content — use 'message_body', NOT 'body'>"
  }
}
```

The `params` must be pre-filled by extracting real data from the email content. Do not leave params empty. Parameter names are case-sensitive — use exactly as shown above.

For the full list of available actions and params, read `{baseDir}/references/actions.md`

### Step 4: Store analysis

Immediately after analysis, store results for **every** email (matched or not):

```
POST /emails/{messageId}/mark-processed
{
  "summary": "<one-line summary>",
  "intent": "<intent>",
  "matched_rules": [{"rule_id": "<id>", "rule_name": "<name>"}],
  "suggested_actions": [{"app": "<app>", "action": "<action>", "label": "<label>"}],
  "actions_taken": []
}
```

For unmatched emails, use empty arrays for `matched_rules`, `suggested_actions`, and `actions_taken`.

Do this **before** outputting the report. The analysis must be persisted regardless of what the user does next.

### Step 5: Report

Output the results to the user. Split emails into two groups: matched first, then unmatched. You **must** use the exact formats below.

---

#### Branch A: Matched emails (hit a rule) — one block per email

For each email that matched a rule, output one block:

```
📌 [Client email] <sender name> sent an email

<one-sentence summary with key details: numbers, dates, names, decisions>

Suggested action: <action label>
[✓ Create] [✗ Skip] [→ View details]
```

`[Client email]` is a fixed label. Output it literally. Do not replace it.

**Example:**

📌 [Client email] David Kim sent an email

Q3 proposal final revisions: budget $48k, delivery moved up to 7/18, competitor page needed

Suggested action: Create task in Notion
[✓ Create] [✗ Skip] [→ View details]

---

#### Branch B: Unmatched emails (no rule hit) — one combined digest block

For all emails that matched NO rules, combine into a single digest block:

```
☀️ Email Digest · <date>

<N> emails pending:
• <Sender>: <one-line description>
• <Sender>: <one-line description>

[→ Open processing page] (link valid for 24h)
```

Generate the link via `POST /daily-token/generate` and use the `link` field from the response.

**Example:**

☀️ Email Digest · Apr 14

3 emails pending:
• Sarah Lee: Asking about next week's schedule
• GitHub: PR #142 awaiting review
• Product Hunt: Daily featured picks

[→ Open processing page] (link valid for 24h)

---

## What NOT to do

- Do not output plain-text summaries like "I found 3 emails, 2 matched rules..."
- Do not use markdown tables or numbered lists instead of the templates above
- Do not skip Step 5 and just describe what you did
- Do not combine matched and unmatched into a single list — they use different formats and serve different purposes
- Do not repeat emails that were already processed in a previous heartbeat cycle
- Do not deviate from the English templates — follow the templates exactly
