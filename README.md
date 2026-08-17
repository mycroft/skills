# Personal Skills

A personal library of Claude Code skills, packaged as a plugin.

## Layout

```
.claude-plugin/
  plugin.json       # plugin manifest
  marketplace.json  # so this repo can be added as a marketplace
skills/
  <skill-name>/
    SKILL.md        # required
    ...             # supporting files, only for tools or heavy reference
```

## Install

### As a Claude Code plugin

```
/plugin marketplace add ~/dev/skills
/plugin install personal-skills@personal-skills-marketplace
```

### With the `skills` CLI (cross-agent)

[`npx skills`](https://github.com/vercel-labs/skills) is the CLI for the open agent skills
ecosystem — it installs skills into Claude Code and ~75 other agents from a repo, URL, or local
path. It operates on individual skills rather than on Claude Code plugins, so it picks the
`skills/*/SKILL.md` entries out of this repo:

```bash
npx skills add ./                 # install from this checkout (project scope)
npx skills add ./ -g              # user-wide: ~/.claude/skills/
npx skills add ./ --list          # show what's available without installing
npx skills update                 # update installed skills to latest
npx skills list                   # what's installed where (alias: ls)
npx skills remove editing-skills  # uninstall
```

Default install is a symlink to a canonical copy, so edits here take effect immediately across
every agent; pass `--copy` if symlinks aren't usable. Scope is project (`./<agent>/skills/`) unless
you pass `-g` for the user directory. Target specific agents with `-a claude-code`.

## Dependency: superpowers

Skills here reference [obra/superpowers](https://github.com/obra/superpowers) skills by name
(e.g. `superpowers:writing-skills`). Install it for those references to resolve — either as a
plugin:

```
/plugin marketplace add obra/superpowers
/plugin install superpowers@superpowers-marketplace
```

or as skills:

```bash
npx skills add obra/superpowers -g
```

Note the plugin route is what upstream supports and tests: superpowers bootstraps itself through its
`using-superpowers` skill at session start, and its docs treat `npx skills` wrapping as a shim rather
than a first-class integration.

## Specification

Skill format follows the [Agent Skills specification](https://agentskills.io) — the shared standard
behind `SKILL.md` and the `skills` CLI. See
[agentskills.io/specification](https://agentskills.io/specification) for every supported frontmatter
field; `name` and `description` are the only required ones.

## Skills

| Skill | Use when |
|---|---|
| `editing-skills` | Changing, fixing, or extending a skill that already exists |

## Conventions

Skills follow `superpowers:writing-skills`:

- Frontmatter carries `name` + `description`; the description states **triggering conditions**, never a
  summary of the workflow (a summary becomes a shortcut agents take instead of reading the skill).
- Cross-reference other skills by name with an explicit marker
  (`**REQUIRED SUB-SKILL:** Use superpowers:writing-skills`). Never `@`-link them — that force-loads
  the file and burns context.
- The Iron Law: no skill, and no skill *edit*, without a failing subagent test first.
