---
name: outline
description: Use when reading, searching, writing, or updating anything in the user's Outline wiki at outline.iop.cx — "check the wiki", "look it up in Outline", "write this up in Outline" — or when an outline.iop.cx URL needs resolving.
---

# Outline

Personal wiki at `https://outline.iop.cx`. MCP endpoint `https://outline.iop.cx/mcp` — streamable
HTTP, stateless (no `initialize` handshake needed), Bearer token from `pass mcp/outline`.

## Pick the access path

1. **Native MCP tools already present** (`list_documents`, `fetch`, `create_document`, possibly
   prefixed `mcp__outline__*`) — use them and stop reading about the script.
2. **Otherwise** use `scripts/outline-mcp` from this skill directory. Same endpoint, same tool names,
   same arguments. Resolve its absolute path once, then reuse it.
3. **Registering the endpoint natively** is a one-time setup step per agent — only when the user asks
   for it. See `reference/setup.md`.

Never put the token on a command line or in a file. The script reads `pass mcp/outline` itself
(or `$OUTLINE_MCP_TOKEN` when already exported).

## Using the script

```bash
outline-mcp tools                                        # names + descriptions
outline-mcp schema update_document                       # exact input schema
outline-mcp call list_documents '{"query":"borg","limit":5}'
outline-mcp call create_document - <<'JSON'              # '-' reads arguments from stdin
{"collectionId":"...","title":"...","text":"body\n"}
JSON
```

Read `schema <tool>` before the first call to a tool — don't guess argument names. Output is one JSON
object per content block, one per line; `fetch` on a document prints metadata JSON, then the markdown
body. `list_documents` results are wrapped: `{"breadcrumb":…,"document":{…}}` (plus `"context"` for
search hits).

## Tools

- **Read** — `list_collections`, `list_documents` (full-text search with `query`, recents without),
  `list_collection_documents` (full hierarchy of one collection), `fetch`
  (`document|collection|user|attachment|template`; `id` accepts an outline.iop.cx URL, or
  `current_user`), `list_templates`, `list_users`, `list_comments`.
- **Write** — `create_document`, `update_document`, `move_document`, `delete_document`,
  `restore_document`, `create_collection`/`update_collection`/`delete_collection`,
  `create_comment`/`update_comment`/`delete_comment`, `create_attachment`.

## Server rules that bite

- Document markdown **must not start with an H1** — the title is a separate field. Start with body
  text or an H2.
- To edit existing content use `update_document` with `editMode:"patch"` and `findText` copied
  verbatim from the document's markdown. The default `replace` overwrites the whole document and
  destroys formatting markdown cannot express (highlights, comments, table widths).
- Mentions are `@[Name](mention://user/<id>)`, ids from `list_users`.
- `list_templates` already returns template bodies; pass `templateId` to `create_document` to use one
  unchanged.

## Before writing

Writes land live in a shared wiki. Search first so you don't create a duplicate, and confirm the
target collection or parent document with the user before creating. Never call a `delete_*` tool
unless the user named that document — archiving via the UI is their call, not yours.
