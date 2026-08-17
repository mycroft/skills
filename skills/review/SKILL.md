---
name: review
description: Use when reviewing code, a diff, or a pull request, before approving or merging, or when asked to give feedback on someone else's change.
---

# Review

## Overview

A review is a ranked list of defects you can demonstrate, plus a verdict. Anything you cannot ground in the lines under review is a question, not a finding.

## Scope

The Summary is one sentence: the file(s) changed, what the change does, and the headline judgement. Nothing else belongs in it.

Everything after the Summary is a finding about a line you can point to.

- If the change exceeds 400 lines, flag it and suggest splitting. Below that, say nothing about size.
- If behavior changed and no test covers it, that is a finding. If tests are present, review them like any other code.

## Severity: assign by test

| Label | Test that must pass to use it |
|--------------|--------------------------------------------------------------|
| `[blocking]` | You can name the concrete input or state that yields a wrong result, data loss, or an auth/security bypass — and the cause is visible in the change |
| `[important]`| Correct as written, but breaks or misleads under a condition you can state |
| `[nit]`      | No behavior change — naming, style, readability |
| `[question]` | Answering it needs information you do not have |

**If you cannot write the input that triggers the failure, it is not `[blocking]`.** Downgrade it to `[important]`, or ask it as `[question]`. A concern about code you cannot see is always `[question]`, never a finding.

## What to examine

- **Correctness** — edge cases, off-by-one, null/undefined, type coercion, race conditions
- **Security** — input validation, injection, authorization checks, secrets, path traversal, non-constant-time comparison of secrets
- **Performance** — N+1 access, unbounded scans, missing pagination, uncleaned timers/listeners/subscriptions
- **Maintainability** — unclear names, duplicated logic, magic numbers, over-complex functions

## Output

```
## Summary
<file(s)> — <what the change does>. <headline judgement>.

## Findings
[severity] One-line claim.
> evidence: the line(s) from the change
> impact: what breaks, and the smallest fix

## Verdict
[ ] Approve   [ ] Comment   [ ] Request changes
```

Order findings hardest-hitting first, in one list — the severity label carries the weight, so there is no separate "required" section to fill.

Every finding carries its evidence line. A finding with no line from the change to point at does not ship.

Check exactly one verdict box and give one sentence of why.

`## Strengths` is optional. Include it only for something introduced *by this change* that is a decision worth repeating — never the change's size, and never code that already existed. If nothing qualifies, leave the section out.

## Feedback principles

Describe the failure, not the author. Give the mechanism, not the verdict alone:

```
[weak]   "This is wrong."
[strong] "Concurrent writes both read the old value here — the second overwrites the first."

[weak]   "Rename this."
[nit]    "`userCount` reads better than `uc`."

[weak]   "You need error handling."
[question] "If this call rejects, should it bubble up or be swallowed here?"
```

## Common mistakes

- Padding `Strengths` to fill the section — praising the diff's size, or code it never touched
- Calling something `[blocking]` without a reproducing input
- Spending Summary on what you could not see instead of on what the change does
- Filing speculation about unseen code as a finding rather than a `[question]`
