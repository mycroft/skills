# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A published collection of Agent Skills (`github.com/mycroft/skills`, public). There is no build, no
runtime, and no test suite — the deliverable is the markdown itself. "Correct" means: valid against
the [Agent Skills spec](https://agentskills.io/specification), and portable across agents.

## Distribution loop

This repo is the source of truth; it is consumed remotely, not from this checkout.

```
skills/<name>/SKILL.md  →  git push  →  npx skills add mycroft/skills
                                     →  ~/.agents/skills/<name>/   (installed copy)
                                     →  ~/.claude/skills/<name>    (symlink)
```

`~/.agents/.skill-lock.json` records each installed skill's origin and a folder hash. Consequences:

- Editing `~/.agents/skills/<name>/` edits the *installed* copy — that change is not in the repo and
  the next `npx skills update` overwrites it. Author here, push, then propagate (below).
- `~/.agents/skills/` also holds skills from other sources (`obra/superpowers`, `mattpocock/skills`,
  …). Not everything installed belongs to this repo — check the lock file's `source` before assuming
  a skill lives here.
- The installed copy reflects GitHub, not this checkout. `origin/main` has been force-pushed before,
  so `git fetch` and check `git status -sb` before starting work; if
  `diff ~/.agents/skills/<name>/SKILL.md skills/<name>/SKILL.md` shows changes you didn't make, the
  checkout is stale.

Propagate from outside the repo. Inside it, the CLI treats `skills/` as project scope, and
`update -y` refreshes those copies instead of the installed ones:

```sh
cd ~
npx skills update <name> -g -y                         # existing skill
npx skills add mycroft/skills -g -s <name> -a '*' -y    # new skill: update ignores it
npx skills remove -g -s <name> -y                      # deleted skill
npx skills list -g                                     # Source should read mycroft/skills
```

The README also documents `npx skills add ./` (a symlink to this checkout) and a Claude Code plugin
install via `.claude-plugin/`. Neither is how the skills are installed here.

## Layout

```
.claude-plugin/
├── plugin.json         # plugin manifest
└── marketplace.json    # lets the repo be added as a plugin marketplace
skills/
└── <skill-name>/
    ├── SKILL.md        # required
    ├── scripts/        # optional: executable code the agent runs (API/MCP wrappers)
    └── reference/      # optional: heavy docs loaded on demand (API, setup)
```

Flat namespace under `skills/`; no nesting or categories. Directory name must equal frontmatter
`name` and is what the user types as `/<skill-name>`. Skills must live under `skills/`: the plugin
loader reads nothing else. Supporting docs go in `reference/` (singular) — that is what the existing
skills use.

## Frontmatter: stick to the six portable fields

`name`, `description`, `license`, `compatibility`, `metadata`, `allowed-tools`.

`name` and `description` are required; the rest are rarely needed. Every skill here uses those two;
only `coding-with-restraint` adds `license`.

**Do not use Claude Code-only fields** (`argument-hint`, `arguments`, `when_to_use`, `context`,
`agent`, `model`, `effort`, `paths`, `hooks`, `disable-model-invocation`, `user-invocable`,
`disallowed-tools`, `shell`). They work in Claude Code but hard-fail packaging and upload for
claude.ai and the Skills API:

```
Unexpected key(s) in SKILL.md frontmatter: argument-hint. Allowed properties are: ...
```

The lock file's `lastSelectedAgents` shows these skills are installed into Codex, Cursor, Gemini CLI,
opencode, Zed and others — same reason to stay on the spec. Claude Code-only *body* features
(`` !`command` `` injection, `$ARGUMENTS`) are equally non-portable; avoid them unless a skill is
deliberately Claude Code-specific, in which case say so in `compatibility`.

Constraints: `name` ≤ 64 chars, lowercase `a-z0-9-`, no leading/trailing or doubled hyphens.
`description` ≤ 1024 chars.

## Writing a description

The description is the only thing an agent sees before deciding to load the skill, so it carries the
whole discovery burden. Start with "Use when…" and state only the triggering conditions — concrete
situations, symptoms, and the words a user would actually type.

```yaml
# thin — no trigger, agent can't tell when this applies
description: Rust coding rules and best practices

# good — triggers only, including the URL a user would paste (from skills/gitea)
description: Use when reading or acting on repositories, issues, pull requests, releases, or
  Actions runs on the user's own Gitea instance at git.mkz.me — including resolving a git.mkz.me
  URL or looking up code the user hosts there.
```

Third person, no first person. Never summarize the skill's workflow in the description — agents
follow the summary instead of reading the body.

## Body

Keep `SKILL.md` under ~500 lines / 5k tokens; existing skills sit at roughly 350–850 words. Move
anything longer into `reference/` and link it with a relative path one level deep — those files load
only when needed.

Prefer tables and numbered steps for reference material. Skills are reusable procedure, not a
narrative of how a problem was solved once.

Cross-reference other skills by name with an explicit marker
(`**REQUIRED SUB-SKILL:** Use superpowers:writing-skills`); never `@`-link them, which force-loads
the file. Skills here depend on [obra/superpowers](https://github.com/obra/superpowers) for those
references to resolve.

## Changing a skill

No skill, and no skill edit, without a failing subagent test first. For an existing skill, follow the
`editing-skills` skill, which builds on `superpowers:writing-skills`.

## Verify before committing

```sh
npx --yes skills-ref@latest validate ./skills/<name>     # frontmatter + naming
for d in skills/*/; do npx --yes skills-ref@latest validate "$d"; done
```

Every skill under `skills/` currently validates clean; keep it that way.

## Adding a skill — checklist

1. `skills/<name>/SKILL.md` with spec frontmatter.
2. Add a row to the README **Skills** table, kept alphabetical:
   `` | `<name>` | <when to use it, as a gerund phrase> | ``. The table is maintained by hand and
   drifts easily.
3. Validate.
4. Push, then `add` it to the agents (see Distribution loop) — `update` won't pick up a new skill.

## Commits

Conventional commits, scoped by skill name: `feat(grafana): add observability access skill`.
Lowercase title, no trailing period, ≤ 50 chars; body explains *why* in bullets. A skill and its
README row land as two commits: `feat(<name>): …`, then `docs(readme): add <name> to the skills
table`.

Every commit in the history carries a `Co-Authored-By: Claude …` trailer, and `skills/commit` does
not forbid one.

## References

- [agentskills.io/specification](https://agentskills.io/specification) — the format spec
- [code.claude.com/docs/en/skills](https://code.claude.com/docs/en/skills) — Claude Code's superset
  and the portability table
- [skills.sh](https://skills.sh/) — the `npx skills` installer
- [obra/superpowers](https://github.com/obra/superpowers) — `writing-skills`, which the conventions
  here follow
