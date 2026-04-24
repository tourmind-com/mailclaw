# MailClaw Heartbeat

This runs periodically to present the user with new emails. Email analysis (intent, summary, rule matching) is handled server-side — the server receives Gmail push notifications via Pub/Sub, analyzes each email, and stores the results automatically. This heartbeat only needs to fetch the pre-analyzed results and present them.

## Steps

Follow these steps **in order**. Do not skip any step. Do not deviate from the output format.

Before starting:

1. Read `{baseDir}/config.json` to get the `api_key` for API calls. If `config.json` is missing, reply `HEARTBEAT_FAIL: config.json not found` and stop.
2. Output this line so the user knows the heartbeat file was loaded:

```
🫀 MailClaw Heartbeat running...
```

### Step 1: Fetch unprocessed emails

```
GET /emails?unprocessed_only=true
```

If the response contains **zero** emails → reply `HEARTBEAT_OK` and stop. Do not proceed.

Each email already includes server-side analysis fields: `summary`, `intent`, `matched_rules`, and `suggested_actions`. Use these directly — do not re-analyze.

### Step 2: Report

Output the results to the user. Split emails into two groups based on the server-provided `matched_rules` field: matched first, then unmatched. You **must** use the exact formats below.

When presenting suggested actions, use the `suggested_actions` from the server response directly.

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

When the user responds:
- **✓ Create** → call `POST /actions/execute` with `{app, action, params}` from the email's `suggested_actions` to trigger execution
- **✗ Skip** → skip this action and move on
- **→ View details** → fetch full email via `GET /gmail/messages/{id}` and display it

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
🔑 Verification code: <code>
```

Generate the link via `POST /daily-token/generate` and use the `link` and `verify_code` fields from the response. Always display the verification code — the user needs it to access the processing page. This endpoint is idempotent per day — morning and evening digests share the same token, code, and link.

**Example:**

☀️ Email Digest · Apr 14

3 emails pending:
• Sarah Lee: Asking about next week's schedule
• GitHub: PR #142 awaiting review
• Product Hunt: Daily featured picks

[→ Open processing page] (link valid for 24h)
🔑 Verification code: 847291

---

## What NOT to do

- Do not re-analyze emails — use the server-provided `summary`, `intent`, `matched_rules`, and `suggested_actions` fields as-is
- Do not call `mark-processed` — the server handles this automatically
- Do not output plain-text summaries like "I found 3 emails, 2 matched rules..."
- Do not use markdown tables or numbered lists instead of the templates above
- Do not skip Step 2 and just describe what you did
- Do not combine matched and unmatched into a single list — they use different formats and serve different purposes
- Do not repeat emails that were already processed in a previous heartbeat cycle
- Do not change the template structure (labels like `📌 [Client email]`, `☀️ Email Digest`, section headings) — keep them in English as defined above. Dynamic content (email summaries, sender descriptions) should follow the original language of the email.
