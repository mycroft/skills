# Gitea REST cheat sheet — git.mkz.me (1.27.2)

Every path below is verified live on this instance. Prefix `/api/v1` is optional with `gitea-api`.
`{R}` stands for `{owner}/{repo}`.

## Why not MCP or tea

- **No MCP server.** `/mcp`, `/api/mcp`, `/api/v1/mcp` all 404 on 1.27.2. The separate `gitea-mcp`
  project would mean installing and running another Go process in front of the same REST API — a
  worse deal than calling it directly. Re-probe `/api/mcp` after a Gitea upgrade.
- **`tea` 0.14.0** is installed but refuses to run without a persisted login (`Error: no available
  login`); `GITEA_SERVER_URL` / `GITEA_SERVER_TOKEN` are only read by `tea login add`, which writes
  the token in cleartext under `$XDG_CONFIG_HOME/tea`. For the user's own interactive use:
  `tea login add --name mkz --url https://git.mkz.me --token "$(pass mcp/gitea | head -1)"`.

## Read

| What | Path |
|---|---|
| current user | `/user` |
| repos the token can see | `/user/repos` (74 here — page it) |
| repo search | `/repos/search` `q=`, `topic=`, `private=` |
| one repo | `/repos/{R}` |
| file, plain text | `/repos/{R}/raw/{path}` `ref=` |
| file, base64 JSON | `/repos/{R}/contents/{path}` |
| recursive file list | `/repos/{R}/git/trees/{ref}` `recursive=1` |
| commits | `/repos/{R}/commits` `sha=`, `path=`, `stat=false` |
| branches / tags | `/repos/{R}/branches`, `/repos/{R}/tags` |
| issues+PRs, all repos | `/repos/issues/search` `state=`, `type=issues|pulls`, `q=`, `labels=` |
| issues+PRs, one repo | `/repos/{R}/issues` `state=`, `type=`, `q=` |
| one PR / its diff files | `/repos/{R}/pulls/{n}`, `/repos/{R}/pulls/{n}/files` |
| comments | `/repos/{R}/issues/{n}/comments` |
| labels / milestones | `/repos/{R}/labels`, `/repos/{R}/milestones` |
| releases | `/repos/{R}/releases` |
| Actions | `/repos/{R}/actions/runs`, `/repos/{R}/actions/tasks` |
| notifications | `/notifications` `all=`, `status-types=` |
| orgs / starred | `/user/orgs`, `/user/starred` |

## Write

| What | Call |
|---|---|
| open issue | `post /repos/{R}/issues '{"title":…,"body":…}'` |
| edit / close issue | `patch /repos/{R}/issues/{n} '{"state":"closed"}'` |
| comment | `post /repos/{R}/issues/{n}/comments '{"body":…}'` |
| open PR | `post /repos/{R}/pulls '{"head":…,"base":…,"title":…}'` |
| merge PR | `post /repos/{R}/pulls/{n}/merge '{"Do":"merge"}'` |
| release | `post /repos/{R}/releases '{"tag_name":…,"name":…}'` |

## jq recipes

```bash
# issue/PR one-liners
gitea-api get /repos/issues/search state=open limit=50 \
  | jq -r '.[] | "\(.repository.full_name)#\(.number) \(if .pull_request then "PR" else "issue" end) \(.title)"'

# repos by last push
gitea-api get /user/repos limit=50 | jq -r 'sort_by(.updated_at) | reverse | .[] | "\(.updated_at[0:10]) \(.full_name)"'

# open PRs waiting on review
gitea-api get /repos/mycroft/k8s-home/pulls state=open limit=50 \
  | jq -r '.[] | "#\(.number) \(.head.ref) → \(.base.ref)  \(.title)"'

# files touched by a PR
gitea-api get /repos/mycroft/k8s-home/pulls/1527/files limit=50 | jq -r '.[].filename'

# last Actions run per workflow
gitea-api get /repos/mycroft/k8s-home/actions/runs limit=20 \
  | jq -r '.workflow_runs[]? // .[] | "\(.status) \(.conclusion // "") \(.display_title // .name)"'
```

## Gotchas

- `limit` caps at 50 server-side; `X-Total-Count` (echoed on stderr) is the real size.
- Issue *numbers* are per-repo (`.number`); `.id` is global — the paths take `.number`.
- `state` defaults to `open` everywhere; pass `state=all` when auditing.
- The token is admin-scoped, so destructive calls will succeed. There is no undo for repo or release
  deletion.
