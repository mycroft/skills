---
name: daily-recap
description: Use when the user asks for a daily recap, a stand-up summary, what they are working on today, the status of their tasks, or which of their pull requests need review or merging — including drafting a message that asks colleagues for reviews.
---

# Daily Recap

Builds the user's daily recap: a review-request message ready to post, then a short status per task
with the next actions. Read-only: never post, merge, approve, or change anything.

## 1. Resolve the sources

The recap uses exactly these four sources. Take each one's location from the context (CLAUDE.md,
AGENTS.md, the agent's memory) and use the fallback only when the context names none:

| Source | From the context | Fallback |
|---|---|---|
| Issue tracker | The tracker it names (Linear, Jira, GitHub Issues, …), through the agent's MCP server or CLI for it | Skip it |
| Notes vault | The vault path it names | Obsidian's open vault: the `"open": true` entry in `obsidian.json` (`~/.config/obsidian/` on Linux, `~/Library/Application Support/obsidian/` on macOS, `%APPDATA%\obsidian\` on Windows) |
| Agent memory | The agent's own persistent memory | — |
| Pull requests | The GitHub org or repos it names | Repos owned by the user's own account |

Only CLAUDE.md, AGENTS.md, and memory can widen the PR scope. An org seen in the vault or in the
user's recent PRs does not: with no org named, the recap covers the user's own repos, even if that
leaves the review request empty.

A source that is unreachable (no connector, not authenticated, path missing) is skipped and listed
at the end of the recap. Do not set it up mid-recap.

## 2. Collect the active tasks

- **Tracker:** issues assigned to the user in a started state (in progress, in review). Skip
  backlog, todo, and done.
- **Vault:** only a note dedicated to tasks, such as `Tasks.md` or `Current work.md`. Other notes
  are background for a task, not tasks.
- **Memory:** entries describing ongoing work.

The same work found in two sources is one task. Name it by its tracker title and key, or by its note
heading.

## 3. Match and classify the PRs

List the user's open PRs in scope, then read each one's state:

```sh
gh search prs --author @me --state open --owner <org-or-user> --json url,title,repository
gh pr view <url> --json title,body,headRefName,isDraft,reviewDecision,mergeable,mergeStateStatus,statusCheckRollup
```

Attach each PR to a task through the tracker's linked PRs, or the task key in its branch, title, or
body. A PR that matches no task but names an issue key forms a task of its own: the key plus a short
topic taken from its PRs, e.g. `OpenTofu migration (PLA-859)`. Only PRs with no key go under
**Other**.

Classify each PR; the first matching row wins:

| PR state | Bucket | Next action |
|---|---|---|
| `isDraft` | in progress | Finish it and mark it ready |
| A check in `statusCheckRollup` failed | fix | Fix the failing check |
| `mergeable: CONFLICTING` | fix | Rebase |
| `reviewDecision: CHANGES_REQUESTED` | fix | Address the review |
| `reviewDecision: APPROVED` | merge | Merge (update the branch first if `mergeStateStatus: BEHIND`) |
| Anything else | needs review (blocked) | Goes in the review request |

## 4. Write the recap

The answer is these three parts, in this order, and nothing else.

**The review request.** Only the *needs review* PRs, grouped by task, in a code block for the user
to post, in exactly this shape:

```
👋 Hey, I would like a quick review for those:
<Task name>:
- <PR URL> ~ <what the PR does, under 10 words>
- <PR URL> ~ <what the PR does, under 10 words>
<Task name>:
- <PR URL> ~ <what the PR does, under 10 words>
Thanks !
```

If no PR needs review, write "No PR waiting for review." in its place.

**One block per task**, most urgent first:

```
**<Task name>** (<tracker key>): <status in a few words>
- <action starting with a verb and naming the PR, e.g. "Merge tf-aws#2589">
- 🚫 **Blocked: <what it waits on>, <who or what unblocks it>**
```

- At most three bullets per task, each something the user can do today.
- Every blocked item (waiting on a review, another team, another PR, or a tracker "blocked" status)
  uses the 🚫 **Blocked:** form.
- A task with nothing to do today gets "nothing to do today" as its status and no bullets.

**One closing line** naming the sources that were skipped, if any.
