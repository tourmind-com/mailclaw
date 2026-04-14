# HubSpot Actions

| Tool Name | Description |
|-----------|-------------|
| `HUBSPOT_CREATE_CONTACT` | Create a new contact |
| `HUBSPOT_CREATE_CONTACT_FROM_NL` | Create contact from natural language |
| `HUBSPOT_UPDATE_CONTACT` | Update contact properties |
| `HUBSPOT_ARCHIVE_CONTACT` | Archive a contact |
| `HUBSPOT_CREATE_COMPANY` | Create a single company |
| `HUBSPOT_ARCHIVE_COMPANY` | Archive a company |
| `HUBSPOT_CREATE_DEAL` | Create a single deal |
| `HUBSPOT_CREATE_DEAL_FROM_NL` | Create deal from natural language |
| `HUBSPOT_UPDATE_DEAL` | Update deal properties |
| `HUBSPOT_CREATE_NOTE` | Create a CRM note |
| `HUBSPOT_CREATE_TICKET` | Create a ticket |
| `HUBSPOT_ARCHIVE_TICKET` | Archive a ticket |
| `HUBSPOT_CREATE_CAMPAIGN` | Create a campaign |
| `HUBSPOT_CREATE_OBJECT_ASSOCIATION` | Create association between records |

## HUBSPOT_CREATE_CONTACT params

```json
{
  "email": "contact@example.com",
  "firstname": "John",
  "lastname": "Doe",
  "company": "Acme Corp",
  "phone": "+1234567890",
  "jobtitle": "Engineer"
}
```

No fields strictly required, but `email` is the unique identifier.

## HUBSPOT_CREATE_DEAL params

```json
{
  "dealname": "New Deal",
  "pipeline": "default",
  "dealstage": "appointmentscheduled",
  "amount": "10000",
  "closedate": "2026-12-31"
}
```

Note: Deal name field is `dealname` (one word, no underscore).

## HUBSPOT_CREATE_NOTE params

```json
{
  "hs_timestamp": "2026-04-14T10:00:00Z",
  "hs_note_body": "Note content here",
  "hubspot_owner_id": "owner-id",
  "associations": [
    {"to": {"id": "contact-id"}, "types": [{"associationCategory": "HUBSPOT_DEFINED", "associationTypeId": 202}]}
  ]
}
```

Required: `hs_timestamp` (ISO-8601 UTC or Unix milliseconds).
Note body field is `hs_note_body`, NOT `body`.
