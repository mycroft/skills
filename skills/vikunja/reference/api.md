# Vikunja REST cheat sheet — todo.iop.cx (v2.5.0)

Swagger 2.0 spec at `/api/v1/docs.json` (425 KB, 126 paths); browsable at `/api/v1/docs`. Pull the
spec when you need a body schema: it is the authority, this file is the shortcut. Paths below are
verified live. The `/api/v1` prefix is optional with `vikunja-api`.

## Why REST and not MCP

Vikunja v2.5.0 ships no MCP server: `/mcp` returns the SPA's `index.html`, `/api/mcp` and
`/api/v1/mcp` 404. Re-probe after an upgrade. There is no first-party CLI either, so the API is the
only route.

## Verb map (inverted — read this first)

| Intent | Call |
|---|---|
| create | `PUT` |
| update | `POST` (full replace — omitted fields are wiped) |
| delete | `DELETE` |

`PATCH` is not used anywhere in the API.

## Read

| What | Path |
|---|---|
| current user | `/user` |
| projects | `/projects` `is_archived=true` to include archived |
| one project | `/projects/{id}` |
| project views (list/gantt/table/kanban) | `/projects/{project}/views` |
| tasks in a view (kanban order) | `/projects/{id}/views/{view}/tasks` |
| tasks, all projects | `/tasks` `filter=`, `s=`, `sort_by=`, `order_by=`, `page=`, `per_page=` |
| one task | `/tasks/{id}` `expand=subtasks` |
| task by project index | `/projects/{project}/tasks/by-index/{index}` |
| labels | `/labels` |
| comments | `/tasks/{taskID}/comments` |
| attachments | `/tasks/{id}/attachments` |
| assignees | `/tasks/{taskID}/assignees` |
| saved filters | `/filters` |
| routes this token may use | `/routes` |
| instance limits | `/info` (`max_items_per_page`, here 50) |

## Write

| What | Call |
|---|---|
| create task | `put /projects/{id}/tasks '{"title":…,"due_date":…,"priority":…}'` |
| update task | `post /tasks/{id} -` with the **whole** object (see recipe below) |
| delete task | `delete /tasks/{id}` |
| create project | `put /projects '{"title":…}'` |
| update project | `post /projects/{id} -` |
| create label | `put /labels '{"title":…}'` |
| attach label | `put /tasks/{id}/labels '{"label_id":N}'` |
| detach label | `delete /tasks/{task}/labels/{label}` |
| comment | `put /tasks/{taskID}/comments '{"comment":…}'` |
| assign | `put /tasks/{taskID}/assignees '{"user_id":N}'` |
| move task between projects | `post /tasks/{id} -` with `project_id` changed |
| bulk edit | `post /tasks/bulk` |

## jq recipes

```bash
# what's due in the next week, soonest first
vikunja-api get /tasks 'filter=done = false && due_date < now+7d' sort_by=due_date order_by=asc per_page=50 \
  | jq -r '.[] | "\(.due_date[0:10])  #\(.id)  \(.title)"'

# everything still open, grouped by project id
vikunja-api get /tasks 'filter=done = false' per_page=50 \
  | jq -r 'group_by(.project_id)[] | "project \(.[0].project_id):", (.[] | "  #\(.id) \(.title)")'

# overdue
vikunja-api get /tasks 'filter=done = false && due_date < now && due_date > "0001-01-02"' per_page=50 \
  | jq -r '.[] | "\(.due_date[0:10]) #\(.id) \(.title)"'

# read-modify-write: complete a task without wiping its other fields
vikunja-api get /tasks/41 | jq '. + {done:true}' | vikunja-api post /tasks/41 -

# reschedule
vikunja-api get /tasks/41 | jq '. + {due_date:"2026-09-01T09:00:00Z"}' | vikunja-api post /tasks/41 -

# label id by title, for filters and attaching
vikunja-api get /labels | jq -r '.[] | "\(.id) \(.title)"'
```

## Filter language

`filter=<expr>`, combined with `&&` / `||`, values unquoted or in double quotes. Date math: `now`,
`now+7d`, `now-30d`, `now/d` (start of day), `now/w`. Verified fields: `done`, `due_date`,
`priority`, `percent_done`, `title` (`like`), `created`, `updated`, `reminders`, `project_id`,
`assignees`, `labels`.

- `labels` matches **ids only** — `labels in 3`. A title (`labels in work`) fails with code 4019 even
  when the label exists.
- `assignees` matches usernames — `assignees in mycroft`.
- `filter_include_nulls=true` keeps tasks whose filtered field is null (default drops them); this is
  why unset due dates disappear from `due_date < …` results.

## Gotchas

- `per_page` caps at 50 server-side; `X-Pagination-Result-Count` / `-Total-Pages` (echoed on stderr)
  are the real totals.
- `0001-01-01T00:00:00Z` is Vikunja's "unset" date, on both read and write.
- Task `identifier` (`#1`) and `index` are per-project; the API paths take the global `id`.
- An API token only reaches the routes listed by `GET /routes` — check there before assuming a 404 is
  a missing object rather than a scope gap.
