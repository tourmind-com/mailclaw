# Actions Index

When you need action names and params for a specific app, read the corresponding file. Only load the files you need — do not load all of them.

| App | File | Common actions |
|-----|------|---------------|
| Gmail | `{baseDir}/references/actions/gmail.md` | `GMAIL_SEND_EMAIL`, `GMAIL_REPLY_TO_THREAD` |
| Google Calendar | `{baseDir}/references/actions/googlecalendar.md` | `GOOGLECALENDAR_CREATE_EVENT`, `GOOGLECALENDAR_QUICK_ADD` |
| Notion | `{baseDir}/references/actions/notion.md` | `NOTION_CREATE_NOTION_PAGE`, `NOTION_INSERT_ROW_DATABASE` |
| Slack | `{baseDir}/references/actions/slack.md` | `SLACK_SENDS_A_MESSAGE_TO_A_SLACK_CHANNEL` |
| HubSpot | `{baseDir}/references/actions/hubspot.md` | `HUBSPOT_CREATE_CONTACT`, `HUBSPOT_CREATE_DEAL`, `HUBSPOT_CREATE_NOTE` |
| Linear | `{baseDir}/references/actions/linear.md` | `LINEAR_CREATE_LINEAR_ISSUE` |

## Rules

- Only use tool names listed in the per-app files. Do not invent action names.
- Parameter names are case-sensitive — use exactly as documented.
- When processing emails, only load the action file for apps referenced by matched rules.
