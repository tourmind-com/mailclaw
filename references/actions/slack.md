# Slack Actions

| Tool Name | Description |
|-----------|-------------|
| `SLACK_SENDS_A_MESSAGE_TO_A_SLACK_CHANNEL` | Send message to a channel |
| `SLACK_CREATE_CHANNEL` | Create a new channel |
| `SLACK_FIND_CHANNELS` | Search channels by criteria |
| `SLACK_LIST_ALL_CHANNELS` | List available conversations |
| `SLACK_FETCH_CONVERSATION_HISTORY` | Get message history from channel |
| `SLACK_FETCH_MESSAGE_THREAD_FROM_A_CONVERSATION` | Get thread replies |
| `SLACK_JOIN_AN_EXISTING_CONVERSATION` | Join a channel |
| `SLACK_INVITE_USERS_TO_A_SLACK_CHANNEL` | Invite users to channel |
| `SLACK_ADD_REACTION_TO_AN_ITEM` | Add emoji reaction to message |
| `SLACK_FIND_USERS` | Search users by email/name |
| `SLACK_LIST_ALL_USERS` | List all workspace users |
| `SLACK_CREATE_A_REMINDER` | Create a reminder |

## SLACK_SENDS_A_MESSAGE_TO_A_SLACK_CHANNEL params

```json
{
  "channel": "channel-id-or-name",
  "text": "Message content (up to 4000 chars)",
  "thread_ts": "optional-parent-message-timestamp"
}
```

Required: `channel`.
