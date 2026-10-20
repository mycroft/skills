---
name: commit
description: Use when the user asks to commit changes, write a commit message, or record work in git.
---

Your task is to help the user to generate a commit message and commit the changes using git.

## Guidelines

- Only generate the message for staged files/changes
- Stage files only as described in Staging below.
- Focus on why the change and not only the change content. If required, ask the user for more explaination.
- Follow the rules below for the commit message.

## Staging

If `git diff --cached` is not empty, the user has already chosen what to commit: commit exactly that
and stage nothing more. Mention any unstaged change that looks like it belongs.

If nothing is staged:

1. Read `git status`, `git diff`, and every untracked file you might add. Decide which files belong
   to the change the user asked to commit.
2. Stage them by explicit path: `git add src/parser.py src/tokenizer.py`. Never `git add .`, `-A`,
   `-u`, a directory, or a wildcard — those stage files you have not read.
3. If a file mixes this change with unrelated edits, stage only the related hunks: write a patch with
   just those hunks and `git apply --cached` it (`git add -p` needs a terminal).
4. If you cannot tell whether a file belongs, leave it unstaged and name it in your reply.

Then, as its own command, run `git diff --cached --stat` and read the output. Every listed path must
belong to the change; unstage any that does not with `git restore --staged <path>`. Run `git commit`
only after this check — never chain `git add` or the check into `git commit` with `&&`.

## Pre-commit Checks

Before generating the commit message, scan the staged diff and **warn the user** (do not block the commit) if you spot any of the following:

- **Secrets or credentials**: API keys, tokens, passwords, private keys, or anything matching patterns like `sk-`, `Authorization: Bearer `, `-----BEGIN`, etc.
- **Syntax issues**: obviously unmatched brackets, braces, or parentheses; incomplete function/class definitions; or truncated files.
- **Debug artifacts**: leftover `console.log`, `print(`, `debugger`, `TODO: remove`, or commented-out blocks that look unintentional.
- **Large unintended files**: binary files, build artifacts, or `node_modules` paths accidentally staged.

State the warning clearly before the commit message, e.g. "Warning: possible secret detected in `config.py` — review before pushing."

## Format

```
<type>(<scope>): <title>          <- 50 characters max, including type and scope

<bullet points summarizing what was updated>
```

`(<scope>)` is optional — use it when one component clearly owns the change.

Count the subject line before committing. If it exceeds 50 characters, cut words from the
title until it fits; do not drop the type or scope to buy room.

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

## Issue References

If the user mentions a related issue, append a trailer at the very end of the message, separated by
a blank line. Which trailer depends on the kind of issue:

| User mentioned | Trailer |
| ---------------------------------------- | -------------------------------------------- |
| A Jira issue (e.g. "this is for ABC-1234") | `JIRA: ABC-1234` |
| Any other issue (e.g. "#42", a tracker URL) | `Refs: #42`, `Fixes: #42`, or `Closes: #42` |

For the non-Jira form, pick by whether this commit resolves the issue:

- `Fixes:` — resolves a bug
- `Closes:` — resolves a feature or task
- `Refs:` — related, but leaves the issue open

`Fixes` and `Closes` auto-close the issue on GitHub and GitLab once the commit reaches the default
branch. If you are not certain this commit resolves the issue, use `Refs`.

```
fix(auth): handle token expiry edge case

- Refresh token is now invalidated on logout

JIRA: ABC-1234
```

If no issue is mentioned, omit the trailer entirely — do not ask for one.

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

When a change fits two types, pick by what it does for someone reading the log:

- Adds a capability that did not exist -> `feat`, even if it also prevents a bug.
- Restores intended behavior of something already shipped -> `fix`.
- Changes no behavior at all -> `refactor`, `style`, `chore`, or `docs`.

