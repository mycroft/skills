---
name: readme
description: Create or update a project README.md with intro, features, getting started, build instructions, and references.
---

When asked to create or update a README, gather context about the project first, then produce or update the file following the structure below.

## Phase 1 — Gather Context

Before writing anything, explore the repository to answer these questions:

- **What does the project do?** Read existing READMEs, package manifests, config files, or entrypoints to understand the purpose and the problem it solves.
- **What language/stack is used?** Check for `package.json`, `Cargo.toml`, `go.mod`, `pyproject.toml`, `build.gradle`, `Makefile`, `Dockerfile`, etc.
- **What are the main features?** Look at the public API surface, CLI commands, exported modules, or documented capabilities.
- **How is the project built and run?** Identify the build tool, test runner, required environment variables, and external dependencies (databases, services, runtimes).
- **Are there existing docs or links?** Look for a `docs/` directory, wiki links, API references, or CI badges.

If any critical information cannot be found in the repository, ask the user before writing.

## Phase 2 — Write or Update the README

Produce the document using the structure below. When updating an existing README, preserve sections that are already accurate — only rewrite or extend what is missing or outdated.

### Section guidance

- **Features**: Only list features that exist; do not describe aspirational capabilities.
- **Prerequisites**: Cover runtime versions, required system tools, external services, and environment variable names (not values).
- **Installation**: If the project has a `.env.example` or similar, include a step to copy it and fill in required values.
- **Tests**: Include relevant flags (e.g. `--watch`, `--coverage`) when they exist.
- **Docker**: Only include this section if the project ships a `Dockerfile` or `docker-compose` file; omit it otherwise.
- **Configuration**: Omit this section entirely if the project has no environment variables — an empty table is worse than no section.
- **References**: Include official docs, API references, design docs, related RFCs, and dependency homepages. Omit generic links (e.g. "GitHub" or "Google").

### Template

```markdown
# <Project Name>

> <One-sentence tagline — what it is and what problem it solves.>

<2–4 sentences expanding on the tagline: context, motivation, and who this project is for.
Mention any notable constraints or design goals that shaped the project.>

## Features

- **<Feature name>** — <one-line description>
- **<Feature name>** — <one-line description>

## Getting Started

### Prerequisites

- <Runtime / language version, e.g. Node.js ≥ 20>
- <Required system tools, e.g. Docker, make>
- <External services, e.g. PostgreSQL 15>
- <Environment variables: list names only>

### Installation

\`\`\`sh
git clone <repo-url>
cd <project-dir>
<package manager install command>
\`\`\`

\`\`\`sh
cp .env.example .env
# Fill in required values
\`\`\`

## Build & Run

### Development

\`\`\`sh
<command to start in development mode>
\`\`\`

### Production Build

\`\`\`sh
<command to build>
<command to start the built artifact>
\`\`\`

### Tests

\`\`\`sh
<command to run the test suite>
\`\`\`

### Docker

\`\`\`sh
<docker build or docker-compose up command>
\`\`\`

## Configuration

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `VAR_NAME` | Yes / No | `value` | What it controls |

## References

- [<Doc title>](<url>) — <one-line description>
```

---

## Guidelines

- **Solve the onboarding problem first.** A new team member should be able to clone the repo, read the README, and have a running instance within minutes. If the build is complex, spell out every step — don't summarize.
- **Be accurate, not aspirational.** Only document what actually exists in the repo. If a feature is planned, omit it.
- **Prefer commands over prose.** When describing something the reader must *do*, show the exact command rather than describing it in words.
- **Pin versions where it matters.** If a specific runtime or tool version is required for the build to succeed, state it explicitly (e.g. `Node.js ≥ 20`, `Go 1.22`).
- **Environment variables are secrets.** List names and descriptions; never include real values or examples of secret material.
- **Keep the intro tight.** Two to four sentences. The "what problem does this solve" must be in the first paragraph — not buried in a later section.
- **Configuration table is required** when the project reads from the environment. An undocumented required variable will block every new contributor.
- **References must be real links.** Do not invent URLs. If you cannot confirm a link exists, write the title only and note that the URL should be added.
- **Update, don't replace.** When a README already exists, compare what's there against the structure above and fill gaps — don't discard accurate content just to reformat.
- **No license or trademark.** Do not include license notices, SPDX identifiers, or trademark statements in the README — these belong in dedicated files (`LICENSE`, `NOTICE`, etc.).
- **No emoji overuse.** Avoid decorating headings, bullet points, or inline text with emojis. Use them only when they carry meaning that plain text cannot convey as clearly.
