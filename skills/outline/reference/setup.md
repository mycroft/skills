# Registering the Outline MCP endpoint

One-time setup, per agent. Only do this when the user asks for it — the bundled
`scripts/outline-mcp` works without any registration.

Endpoint: `https://outline.iop.cx/mcp` (streamable HTTP). Auth: `Authorization: Bearer <token>`,
token in `pass mcp/outline`.

The token must never be written into a config file in plaintext. Both agents below can read it from
an environment variable instead, so the shell session that launches the agent is the only place the
secret exists:

```bash
export OUTLINE_MCP_TOKEN=$(pass mcp/outline)   # or prefix the launch command with it
```

## Claude Code (verified, 2.1.239)

```bash
claude mcp add --transport http outline https://outline.iop.cx/mcp \
  --header 'Authorization: Bearer ${OUTLINE_MCP_TOKEN}'
```

Single quotes matter: `${OUTLINE_MCP_TOKEN}` must reach the config unexpanded — Claude Code expands
it when it connects. Add `-s user` for all projects instead of the current one.

Verify: `OUTLINE_MCP_TOKEN=$(pass mcp/outline) claude mcp list` prints `outline: … ✔ Connected`.
Without the variable the same command reports `Missing environment variables: OUTLINE_MCP_TOKEN` and
a 401 — that is the expected failure, not a broken config.

## Codex CLI (0.149.0)

```bash
codex mcp add outline --url https://outline.iop.cx/mcp \
  --bearer-token-env-var OUTLINE_MCP_TOKEN
```

Writes to `~/.codex/config.toml`; the variable is read from Codex's own environment at connect time,
so export it before launching. `codex mcp list` shows the registration. Not verified against a live
Codex session on this machine (no Codex credentials present) — if it fails, fall back to the script.

## pi (0.84.2)

pi has no MCP client: no `mcp` command, no `mcpServers` settings key, nothing in its dist bundle.
Use `scripts/outline-mcp` — pi's bash tool runs it fine. Re-check `pi --help` for an `mcp` command
before concluding this is still true.
