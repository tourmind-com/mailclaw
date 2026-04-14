# Google Calendar Actions

| Tool Name | Description |
|-----------|-------------|
| `GOOGLECALENDAR_CREATE_EVENT` | Create a new calendar event |
| `GOOGLECALENDAR_QUICK_ADD` | Create event from natural language text |
| `GOOGLECALENDAR_FIND_EVENT` | Search events by text and time range |
| `GOOGLECALENDAR_EVENTS_LIST` | List events within specified criteria |
| `GOOGLECALENDAR_EVENTS_GET` | Retrieve a single event by ID |
| `GOOGLECALENDAR_UPDATE_EVENT` | Full replacement update of an event |
| `GOOGLECALENDAR_PATCH_EVENT` | Partially update specific event fields |
| `GOOGLECALENDAR_DELETE_EVENT` | Remove an event from calendar |
| `GOOGLECALENDAR_FIND_FREE_SLOTS` | Find availability windows |
| `GOOGLECALENDAR_LIST_CALENDARS` | List user's calendars |
| `GOOGLECALENDAR_GET_CURRENT_DATE_TIME` | Get current date/time in a timezone |

## GOOGLECALENDAR_CREATE_EVENT params

```json
{
  "summary": "Meeting title",
  "start_datetime": "2026-04-15T14:00:00",
  "event_duration_hour": 1,
  "event_duration_minutes": 0,
  "timezone": "Asia/Shanghai",
  "description": "Meeting details and agenda",
  "location": "Zoom link or address",
  "attendees": ["email1@example.com"],
  "create_meeting_room": true
}
```

Required: `start_datetime` (ISO 8601: YYYY-MM-DDTHH:MM:SS). Natural language NOT supported.
The event title field is `summary`, NOT `title`.
Use `event_duration_hour` + `event_duration_minutes` for duration, OR `end_datetime`.

## GOOGLECALENDAR_QUICK_ADD params

```json
{
  "text": "Meeting with David on April 15 at 2pm for 1 hour"
}
```
