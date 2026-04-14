# Linear Actions

| Tool Name | Description |
|-----------|-------------|
| `LINEAR_CREATE_LINEAR_ISSUE` | Create a new issue |
| `LINEAR_UPDATE_LINEAR_ISSUE` | Update an existing issue |
| `LINEAR_LIST_LINEAR_ISSUES` | List issues with filters |
| `LINEAR_SEARCH_LINEAR_ISSUES` | Search issues by text |
| `LINEAR_CREATE_LINEAR_COMMENT` | Add comment to an issue |
| `LINEAR_LIST_LINEAR_TEAMS` | List all teams |
| `LINEAR_LIST_LINEAR_PROJECTS` | List all projects |

## LINEAR_CREATE_LINEAR_ISSUE params

```json
{
  "title": "Issue title",
  "team_id": "team-uuid",
  "description": "Detailed description (supports markdown)",
  "priority": 3,
  "due_date": "2026-12-31"
}
```

Required: `title`, `team_id` (UUID). Priority: 0=none, 1=Urgent, 2=High, 3=Normal, 4=Low.
