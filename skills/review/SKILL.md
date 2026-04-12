---
name: review
description: Review code or a pull request. Catches bugs, security issues, and design problems. Gives structured, constructive feedback.
---

When asked to review code or a PR, work through the following phases in order, then produce a structured report.

## Phase 1 — Context

Before reading code:

- Read the PR description and linked issue if available
- Note PR size: if >400 lines, flag it and suggest splitting
- Check whether CI is passing

## Phase 2 — High-Level

- **Architecture**: Does the solution fit the problem? Are dependencies pointing in the right direction? Any God objects, circular dependencies, or layer violations?
- **Design**: Does it follow existing patterns in the codebase? Any obvious over-engineering or under-engineering?
- **Tests**: Is there coverage for new behavior and edge cases?

## Phase 3 — Line-by-Line

For each file, check:

- **Logic & correctness** — edge cases, off-by-one, null/undefined, race conditions
- **Security** — input validation, SQL/command injection, XSS, auth checks, secrets in code, path traversal
- **Performance** — N+1 queries, O(n²) loops, missing pagination, memory leaks (uncleaned listeners/timers/subscriptions)
- **Maintainability** — unclear names, duplicated logic, magic numbers, overly complex functions

## Phase 4 — Output

Write the review using this structure:

```
## Summary
[1-2 sentences: what was reviewed, overall impression]

## Strengths
- [What was done well]

## Required Changes
[blocking] [description]
> [location or example]
> [suggested fix]

## Suggestions
[important] [description — should fix but not blocking]
[nit] [minor improvement — not blocking]
[question] [clarification needed]

## Verdict
[ ] Approve
[ ] Comment (minor suggestions, can merge)
[ ] Request Changes (blocking issues must be addressed)
```

## Severity Labels

| Label        | Meaning                         | Action            |
|--------------|---------------------------------|-------------------|
| `[blocking]` | Must fix before merge           | Request changes   |
| `[important]`| Should fix, discuss if disagree | Comment or block  |
| `[nit]`      | Nice to have, not blocking      | Comment           |
| `[question]` | Need clarification              | Ask               |
| `[praise]`   | Something done well             | Celebrate         |

## Feedback Principles

Be specific and collaborative, not judgmental:

```
[bad]  "This is wrong."
[good] "This could cause a race condition under concurrent writes — consider a mutex here."

[bad]  "Rename this."
[nit]  "Consider `userCount` over `uc` for readability. Not blocking."

[bad]  "You need error handling here."
[good] "What happens if this API call fails? Should it bubble up or be caught here?"
```

Focus on the code, not the author. Praise good work. Only block on things that genuinely matter.
