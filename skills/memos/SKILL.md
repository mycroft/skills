---
name: memos
description: Use when reading, searching, writing, or updating notes in the user's Memos instance at memos.iop.cx — "check my memos", "note this down", "what did I jot about X" — or when a memos.iop.cx URL needs resolving.
---

# Memos

Personal note stream at `https://memos.iop.cx` (Memos 0.30.0). MCP endpoint `https://memos.iop.cx/mcp`
— streamable HTTP; the server hands out an `Mcp-Session-Id` but does not require it, and no
`initialize` handshake is needed. Token from `pass mcp/memos`.

## Pick the access path

1. **Native MCP tools already present** (`memo_list_memos`, `memo_create_memo`, possibly prefixed
   `mcp__memos__*`) — use them and stop reading about the script.
2. **Otherwise** use `scripts/memos-mcp` from this skill directory. Resolve its absolute path once,
   then reuse it.
3. **Registering the endpoint natively** is a one-time setup step per agent — only when the user asks
   for it. See `reference/setup.md`.

Never put the token on a command line or in a file. The script reads `pass mcp/memos` itself (or
`$MEMOS_MCP_TOKEN` when already exported).

## Using the script

```bash
memos-mcp tools                                       # 20 tools, one line each
memos-mcp schema memo_update_memo                     # input schema (-r keeps the $defs bulk)
memos-mcp call memo_list_memos '{"filter":"\"work\" in tags","pageSize":10}'
memos-mcp call memo_create_memo - <<'JSON'            # '-' reads arguments from stdin
{"body":{"content":"note text #tag","visibility":"PRIVATE"}}
JSON
```

Each call prints one JSON object — pipe to `jq`. The tools are generated from the REST API, so the
JSON is the REST payload: `{"memos":[…],"nextPageToken":…}` for lists. `tools/list` works without a
token; every tool call needs one.

## Data model

- A memo is flat markdown with **no title** — `content` is everything. Tags are `#hashtags` inside the
  content; the `tags` field is read-only output.
- `visibility` is `PRIVATE` (default) | `PROTECTED` (signed-in users) | `PUBLIC`.
- Ids: memos are named `memos/{id}`; tools accept either that or the bare `{id}`.
- Comments are memos too: `memo_create_memo_comment`, `memo_list_memo_comments`.

## Traps

- `memo_update_memo` needs `updateMask` (comma-separated field list) alongside `body`, or the update
  is silently a no-op. Read the current memo first — `body.content` replaces the whole note.
- `filter` is CEL over `content`, `creator`, `created_ts`, `updated_ts`, `pinned`, `visibility`,
  `tags`, `has_task_list`, `has_link`, `has_code`, `has_incomplete_tasks`. Tags match as
  `"work" in tags`, never `tags == "work"`. `content.contains()` is case-insensitive.
- `orderBy` uses **`create_time` / `update_time`**; `filter` uses **`created_ts` / `updated_ts`**.
  Mixing them up is an error, not a fallback.
- `pageSize` defaults to 50, caps at 1000; archived notes need `state:"ARCHIVED"`.

## Before writing

Notes land in the user's live stream immediately. Search first so you don't duplicate an existing
note, keep new memos `PRIVATE` unless the user says otherwise, and never call `memo_delete_memo`
unless the user pointed at that memo.
