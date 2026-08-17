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

```
/plugin marketplace add ~/dev/skills
/plugin install personal-skills@personal-skills-marketplace
```

## Dependency: superpowers

Skills here reference [obra/superpowers](https://github.com/obra/superpowers) skills by name
(e.g. `superpowers:writing-skills`). Install it for those references to resolve:

```
/plugin marketplace add obra/superpowers
/plugin install superpowers@superpowers-marketplace
```

## Skills

| Skill | Use when |
|---|---|
| `update-skill` | Changing, fixing, or extending a skill that already exists |

## Conventions

Skills follow `superpowers:writing-skills`:

- Frontmatter is `name` + `description` only; the description states **triggering conditions**, never a
  summary of the workflow (a summary becomes a shortcut agents take instead of reading the skill).
- Cross-reference other skills by name with an explicit marker
  (`**REQUIRED SUB-SKILL:** Use superpowers:writing-skills`). Never `@`-link them — that force-loads
  the file and burns context.
- The Iron Law: no skill, and no skill *edit*, without a failing subagent test first.
