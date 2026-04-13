# skills

My personal collection of agent skills for [Claude Code](https://claude.ai/code) and other AI agents.

## What are skills?

Skills are reusable, modular capabilities you can install into AI agents to extend their behavior with procedural knowledge — custom slash commands, workflows, and domain-specific guidance.

See [skills.sh](https://skills.sh/) for the broader ecosystem and a directory of community skills.

## Installation

Install skills from any GitHub repository using [`npx skills`](https://skills.sh/):

```bash
npx skills add <owner/repo>
```

For example, to install from this repo:

```bash
npx skills add mycroft/skills
```

## Skills

| Skill | Description |
|-------|-------------|
| [commit](skills/commit/SKILL.md) | Commit current changes with a descriptive, conventional message |
| [migration-plan](skills/migration-plan/SKILL.md) | Prepare a structured migration plan with steps, verification, revert paths, and communication |
| [review](skills/review/SKILL.md) | Review code or a PR — catches bugs, security issues, and design problems |
