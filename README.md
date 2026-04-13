# skills

My personal collection of agent skills for [Claude Code](https://claude.ai/code) and other AI agents.

## What are skills?

Skills are reusable, modular capabilities you can install into AI agents to extend their behavior with procedural knowledge — custom slash commands, workflows, and domain-specific guidance. Each skill lives in its own directory as a `SKILL.md` file that instructs the agent on how to approach a specific task.

See [skills.sh](https://skills.sh/) for the broader ecosystem and a directory of community skills.

## Features

- **Conventional commits** — generates structured commit messages with pre-commit safety checks and optional Jira integration
- **Code review** — phased PR review covering architecture, correctness, security, and performance
- **Migration planning** — produces structured migration documents with steps, verification checklists, and revert paths
- **README generation** — creates or updates project READMEs with a consistent, onboarding-focused structure

## Getting Started

### Prerequisites

- [Node.js](https://nodejs.org/) — required by the `npx skills` installer
- [Claude Code](https://claude.ai/code) or another agent that supports skill loading

### Installation

Install skills from this repository using [`npx skills`](https://skills.sh/):

```sh
npx skills add mycroft/skills
```

To install a single skill:

```sh
npx skills add mycroft/skills/<skill-name>
```

## Skills

| Skill | Description |
|-------|-------------|
| [commit](skills/commit/SKILL.md) | Commit current changes with a descriptive, conventional message |
| [migration-plan](skills/migration-plan/SKILL.md) | Prepare a structured migration plan with steps, verification, revert paths, and communication |
| [readme](skills/readme/SKILL.md) | Create or update a project README with intro, features, build instructions, and references |
| [review](skills/review/SKILL.md) | Review code or a PR — catches bugs, security issues, and design problems |

## Adding a Skill

1. Create a directory under `skills/` named after your skill.
2. Add a `SKILL.md` file with YAML frontmatter (`name`, `description`) followed by the skill instructions.
3. Add a row to the table above.

```
skills/
└── my-skill/
    └── SKILL.md
```

`SKILL.md` minimum structure:

```markdown
---
name: my-skill
description: One-line description of what this skill does.
---

Instructions for the agent...
```

## References

- [skills.sh](https://skills.sh/) — skill ecosystem, installer docs, and community directory
- [Claude Code documentation](https://docs.anthropic.com/en/docs/claude-code) — Claude Code CLI reference
- [Conventional Commits](https://www.conventionalcommits.org/) — commit message specification used by the `commit` skill
