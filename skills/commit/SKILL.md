---
name: commit
description: Commit current change with a descriptive message
---

Your task is to help the user to generate a commit message and commit the changes using git.

## Guidelines

- DO NOT add any AI agent ads such as "Generated with [Claude Code](https://claude.ai/code)"
- Only generate the message for staged files/changes
- Don't add any files using `git add`. The user will decide what to add. 
- Focus on why the change and not only the change content. If required, ask the user for more explaination.
- Follow the rules below for the commit message.

## Pre-commit Checks

Before generating the commit message, scan the staged diff and **warn the user** (do not block the commit) if you spot any of the following:

- **Secrets or credentials**: API keys, tokens, passwords, private keys, or anything matching patterns like `sk-`, `Authorization: Bearer `, `-----BEGIN`, etc.
- **Syntax issues**: obviously unmatched brackets, braces, or parentheses; incomplete function/class definitions; or truncated files.
- **Debug artifacts**: leftover `console.log`, `print(`, `debugger`, `TODO: remove`, or commented-out blocks that look unintentional.
- **Large unintended files**: binary files, build artifacts, or `node_modules` paths accidentally staged.

State the warning clearly before the commit message, e.g. "Warning: possible secret detected in `config.py` — review before pushing."

## Format

```
<type>:<space><message title>

<bullet points summarizing what was updated>
```

## Example Titles

```
feat(auth): add JWT login flow
fix(ui): handle null pointer in sidebar
refactor(api): split user controller logic
docs(readme): add usage section
```

## Example with Title and Body

```
feat(auth): add JWT login flow

- Implemented JWT token validation logic
- Added documentation for the validation component
```

## Jira Integration

If the user mentions a related Jira issue (e.g. "this is for ABC-1234"), append a `JIRA: <issue-key>` line at the very end of the commit message, separated by a blank line:

```
fix(auth): handle token expiry edge case

- Refresh token is now invalidated on logout

JIRA: ABC-1234
Co-Authored-By: nobody <nobody@example.com>
```

If no issue is mentioned, omit the line entirely — do not ask for one.

## Rules

* title is lowercase, no period at the end.
* Title should be a clear summary, max 50 characters.
* Use the body (optional) to explain *why*, not just *what*.
* Bullet points should be concise and high-level.

Avoid

* Vague titles like: "update", "fix stuff"
* Overly long or unfocused titles
* Excessive detail in bullet points

## Allowed Types

| Type     | Description                           |
| -------- | ------------------------------------- |
| feat     | New feature                           |
| fix      | Bug fix                               |
| chore    | Maintenance (e.g., tooling, deps)     |
| docs     | Documentation changes                 |
| refactor | Code restructure (no behavior change) |
| test     | Adding or refactoring tests           |
| style    | Code formatting (no logic change)     |
| perf     | Performance improvements              |

