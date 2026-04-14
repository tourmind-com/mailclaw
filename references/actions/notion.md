# Notion Actions

| Tool Name | Description |
|-----------|-------------|
| `NOTION_CREATE_NOTION_PAGE` | Create a new page |
| `NOTION_SEARCH_NOTION_PAGE` | Search pages by title |
| `NOTION_RETRIEVE_PAGE` | Get page properties/metadata |
| `NOTION_UPDATE_PAGE` | Update page properties, icon, cover |
| `NOTION_ARCHIVE_NOTION_PAGE` | Move page to trash or restore |
| `NOTION_GET_PAGE_MARKDOWN` | Get full page content as markdown |
| `NOTION_ADD_PAGE_CONTENT` | Add content blocks to a page |
| `NOTION_ADD_MULTIPLE_PAGE_CONTENT` | Bulk-add content blocks |
| `NOTION_CREATE_DATABASE` | Create a new database |
| `NOTION_FETCH_DATABASE` | Get database structure metadata |
| `NOTION_QUERY_DATABASE` | Query database rows with sorting |
| `NOTION_INSERT_ROW_DATABASE` | Insert a new database row |
| `NOTION_UPDATE_ROW_DATABASE` | Update an existing row |
| `NOTION_FETCH_DATA` | List pages/databases in workspace |

## NOTION_CREATE_NOTION_PAGE params

```json
{
  "title": "Page title",
  "parent_id": "database-or-page-uuid",
  "markdown": "Page content in Notion-flavored Markdown"
}
```

Required: `title`, `parent_id` (UUID format preferred).
Content field is `markdown`, NOT `content`.

## NOTION_INSERT_ROW_DATABASE params

```json
{
  "database_id": "database-uuid",
  "properties": [
    {"name": "Name", "type": "title", "value": "Row title"},
    {"name": "Status", "type": "select", "value": "In Progress"}
  ]
}
```

Required: `database_id`. Property names must match database schema exactly (case-sensitive).
