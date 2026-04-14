# Gmail Actions

| Tool Name | Description |
|-----------|-------------|
| `GMAIL_FETCH_EMAILS` | Retrieve email messages with filtering |
| `GMAIL_FETCH_MESSAGE_BY_MESSAGE_ID` | Fetch a specific message by ID |
| `GMAIL_FETCH_MESSAGE_BY_THREAD_ID` | Retrieve messages from a thread |
| `GMAIL_SEND_EMAIL` | Send a new email |
| `GMAIL_REPLY_TO_THREAD` | Reply to an email thread |
| `GMAIL_FORWARD_MESSAGE` | Forward an existing message |
| `GMAIL_CREATE_EMAIL_DRAFT` | Create an email draft |
| `GMAIL_SEND_DRAFT` | Send an existing draft |
| `GMAIL_ADD_LABEL_TO_EMAIL` | Add/remove labels on a message |
| `GMAIL_MOVE_TO_TRASH` | Move a message to trash |
| `GMAIL_LIST_LABELS` | List all labels |
| `GMAIL_LIST_THREADS` | List email threads |
| `GMAIL_GET_PROFILE` | Get Gmail account profile info |
| `GMAIL_GET_ATTACHMENT` | Retrieve a message attachment |

## GMAIL_SEND_EMAIL params

```json
{
  "recipient_email": "user@example.com",
  "subject": "Email subject",
  "body": "Email content (plain text or HTML)",
  "is_html": false,
  "cc": ["cc@example.com"],
  "bcc": ["bcc@example.com"]
}
```

Note: `recipient_email` (or alias `to`) is the recipient field. At least one of `recipient_email`, `cc`, or `bcc` must be provided. Either `subject` or `body` must be provided.

## GMAIL_REPLY_TO_THREAD params

```json
{
  "thread_id": "19d8a40fbe56cb27",
  "message_body": "Reply content",
  "recipient_email": "user@example.com",
  "is_html": false
}
```

Note: The body field is `message_body` here, NOT `body`.
