---
name: vikunja
description: Use when reading, adding, completing, or rescheduling the user's to-dos in Vikunja at todo.iop.cx — "what's on my list", "add a task", "mark that done", "what's due this week" — or when a todo.iop.cx URL needs resolving.
---

# Vikunja (todo.iop.cx)

Task tracker, Vikunja v2.5.0. **No MCP server** — `/mcp` serves the web app, `/api/mcp` and
`/api/v1/mcp` 404 — so the REST API is the access path. Token (`tk_…`) from `pass mcp/vikunja`, sent
as `Authorization: Bearer`. Interactive API docs: `https://todo.iop.cx/api/v1/docs`
(spec: `/api/v1/docs.json`).

Use `scripts/vikunja-api` from this skill directory; resolve its absolute path once, then reuse it.
Never put the token on a command line or in a file — the script reads `pass mcp/vikunja` itself (or
`$VIKUNJA_TOKEN` when already exported).

## Two traps that lose data

**1. The verbs are inverted: `PUT` creates, `POST` updates.** There is no `PATCH`.

**2. `POST /tasks/{id}` replaces the whole task — every field you omit is wiped.** Sending
`{"title":"new"}` blanks the description, resets the due date to the zero date and priority to 0
(verified). Always read-modify-write:

```bash
vikunja-api get /tasks/41 | jq '. + {done:true}' | vikunja-api post /tasks/41 -
```

## Using the script

```bash
vikunja-api get /projects per_page=50
vikunja-api get /tasks 'filter=done = false && due_date < now+7d' sort_by=due_date order_by=asc
vikunja-api put /projects/6/tasks '{"title":"pay rent","due_date":"2026-08-25T12:00:00Z","priority":3}'
vikunja-api delete /tasks/41
```

The `/api/v1` prefix is optional. Bodies pass through exactly as Vikunja sent them —
compact JSON, so pipe to `jq`. `per_page` caps at **50**; the script echoes `result-count` and `total-pages`
on stderr, so check them before calling a list complete.

## Data model

- Tasks live in a project; today the instance has one, `Inbox` (id 6). Resolve the project with
  `get /projects` rather than assuming an id, and ask the user before inventing a new project.
- Dates are RFC 3339 UTC. **`0001-01-01T00:00:00Z` means unset** — that is what an empty due date
  reads as, and what you post to clear one.
- `done` is a boolean; the server stamps `done_at` itself. `priority` is 0–5, `percent_done` 0–100.
- Labels are objects, not strings: create with `put /labels`, attach with
  `put /tasks/{id}/labels '{"label_id":N}'`.
- Comments: `put /tasks/{id}/comments '{"comment":"…"}'`.

## Filters

`filter=` takes an expression, `&&`/`||`, with date math like `now+7d`, `now/d` (start of day):
`done = false && due_date < now+7d`, `priority >= 3`, `title like "report"`, `created > now-30d`.

- **Labels filter by id, never by title** — `labels in 3`, not `labels in work` (rejected, code 4019).
  Resolve titles through `get /labels` first.
- Assignees do take usernames: `assignees in mycroft`.

`reference/api.md` has the endpoint table and jq recipes.

## Before writing

These are the user's real to-dos. Confirm the target project before creating, echo back what you are
about to change on an existing task, and never `delete` a task or project unless the user named it. The
API exposes no trash or restore route, so treat a delete as final (projects can be archived with
`is_archived` instead).
