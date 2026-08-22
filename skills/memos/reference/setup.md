# Registering the Memos MCP endpoint

One-time setup, per agent. Only do this when the user asks for it — the bundled
`scripts/memos-mcp` works without any registration.

Endpoint: `https://memos.iop.cx/mcp` (streamable HTTP, Memos 0.30.0, MCP protocol 2025-11-25).
Auth: `Authorization: Bearer <token>`, token in `pass mcp/memos`. Tokens come from the Memos UI —
Settings → Access Tokens.

Keep the token out of config files; both agents below read it from an environment variable, so the
shell that launches the agent is the only place it exists:

```bash
export MEMOS_MCP_TOKEN=$(pass mcp/memos | head -1)   # or prefix the launch command with it
```

## Claude Code (verified, 2.1.239)

```bash
claude mcp add --transport http memos https://memos.iop.cx/mcp \
  --header 'Authorization: Bearer ${MEMOS_MCP_TOKEN}'
```

Single quotes matter: `${MEMOS_MCP_TOKEN}` must reach the config unexpanded — Claude Code expands it
at connect time. Add `-s user` for all projects instead of the current one.

**`claude mcp list` saying `✔ Connected` does not mean the token works.** Memos answers `initialize`
and `tools/list` unauthenticated, so a bogus token still shows as connected (verified). Test auth with
a real call instead:

```bash
memos-mcp call auth_get_current_user     # or ask the agent to run the native tool
```

A bad token returns `401 Unauthorized: authentication required` from the tool, not from the transport.

## Codex CLI (0.149.0)

```bash
codex mcp add memos --url https://memos.iop.cx/mcp \
  --bearer-token-env-var MEMOS_MCP_TOKEN
```

Writes to `~/.codex/config.toml`; the variable is read from Codex's own environment at connect time,
so export it before launching. Not verified against a live Codex session on this machine (no Codex
credentials present) — if it fails, fall back to the script.

## pi (0.84.2)

pi has no MCP client: no `mcp` command, no `mcpServers` settings key, nothing in its dist bundle. Use
`scripts/memos-mcp` — pi's bash tool runs it fine. Re-check `pi --help` for an `mcp` command before
concluding this is still true.
