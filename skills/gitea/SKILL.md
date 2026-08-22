---
name: gitea
description: Use when reading or acting on repositories, issues, pull requests, releases, or Actions runs on the user's own Gitea instance at git.mkz.me — including resolving a git.mkz.me URL or looking up code the user hosts there.
---

# Gitea (git.mkz.me)

Self-hosted Gitea 1.27.2. **It has no MCP server** — `/mcp`, `/api/mcp` and `/api/v1/mcp` all 404, so
the REST API is the access path. Token from `pass mcp/gitea`, sent as `Authorization: token <tok>`.

Use `scripts/gitea-api` from this skill directory; resolve its absolute path once, then reuse it.
Never put the token on a command line or in a file — the script reads `pass mcp/gitea` itself (or
`$GITEA_TOKEN` when already exported).

## Choosing the tool

- **`gitea-api`** for anything about issues, PRs, releases, Actions, notifications, or repo metadata.
- **Plain `git`** for code and history — clone or read a local checkout instead of walking the API
  file by file. Clone over SSH; never build an HTTPS URL with the token in it.
- **`tea`** (0.14.0, installed) needs `tea login add`, which persists the token in cleartext under
  `$XDG_CONFIG_HOME/tea`. Env vars alone give "no available login". Don't set it up for agent work —
  suggest it only if the user wants it for their own interactive use.

## Using the script

```bash
gitea-api get /user/repos limit=50 page=2          # k=v args are url-encoded query params
gitea-api get /repos/issues/search state=open type=issues limit=20
gitea-api post /repos/mycroft/foo/issues '{"title":"…","body":"…"}'
gitea-api patch /repos/mycroft/foo/issues/3 - <<'JSON'
{"state":"closed"}
JSON
```

The `/api/v1` prefix is optional. Bodies come back exactly as Gitea sent them — compact JSON, or
plain text for `/raw/` paths — so pipe to `jq`. Objects are fat (~1.5 KB per issue, ~3 KB per repo):
project the fields you need rather than dumping arrays.

## API facts that matter

- `limit` **caps at 50** whatever you ask for. The script echoes `X-Total-Count` on stderr — check it
  before concluding a list is complete, and page with `page=N`.
- `/repos/{o}/{r}/raw/{path}` returns file content as plain text; `/contents/{path}` returns it
  base64-wrapped in JSON. Prefer `raw`.
- Issue endpoints return **PRs as well as issues**. Filter with `type=issues` or `type=pulls`, and
  test `.pull_request != null` when reading mixed results.
- Cross-repo work goes through `/repos/issues/search` (all repos the token can see) and
  `/repos/search?q=`; per-repo paths are `/repos/{owner}/{repo}/…`.
- Recursive file listing: `/repos/{o}/{r}/git/trees/{ref}?recursive=1`.

`reference/api.md` has the endpoint cheat sheet and jq recipes.

## Before writing

This is the user's live instance and the token is admin-scoped, so nothing stops a bad call. Confirm
the target repo with the user before opening, editing, or closing anything, and never call `delete`
on a repo, release, or issue unless they asked for that object by name.
