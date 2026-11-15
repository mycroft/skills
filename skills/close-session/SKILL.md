---
name: close-session
description: Use when the user wraps up, closes, or ends a session or discussion and wants nothing from it lost — "close session", "wrap up", "let's close this", "before I close", "save what we learned".
---

# Close Session

Nothing from the session may be lost when it closes. Knowledge goes to the notes vault, the agent's
memory keeps only what is about the user, the team hears status on the issue, and leftover work is
settled with the user. The vault and memory are written directly; the issue tracker is never written
without the user's yes.

## 1. Resolve the destinations

Take each location from the context (CLAUDE.md, AGENTS.md, the agent's memory). Use the fallback only
when the context names none:

| Destination | From the context | Fallback |
|---|---|---|
| Notes vault | The vault path it names | Obsidian's open vault: the `"open": true` entry in `obsidian.json` — `~/.config/obsidian/`, `~/Library/Application Support/obsidian/` (macOS), `%APPDATA%\obsidian\` (Windows) |
| Issue tracker | The tracker it names, through its MCP server or CLI | None |
| Agent memory | The agent's own persistent memory | None |

An unreachable destination never drops items: vault items go to memory, marked "move to the vault";
tracker drafts go in the report for the user to paste.

## 2. Take stock

Read every source the session produced:

- **The conversation**, including any compacted summary.
- **Files the session wrote outside repos**: scratchpad, plan files, scratch directories. They hold
  findings the conversation may have lost to compaction, and they disappear with the session.
- **Every repo and worktree the session touched**: `git status --short --branch`,
  `git log --oneline --branches --not --remotes`, `git worktree list`.
- **Memory entries the session read or wrote.**

List what is worth keeping: decisions and why, findings and gotchas, procedures carried out,
corrections or preferences the user stated, work done (PRs, commits), promises not kept, follow-ups,
and leftover changes.

If nothing is worth keeping, say so in one line and stop.

## 3. Route every item to one home

| The item is about… | Home |
|---|---|
| How a system works, a decision and its reason, a finding, a procedure, a follow-up | Vault |
| The user and how to work with them: a preference, a correction, where things live | Memory |
| Status the team needs: what was done, PRs, what is left | Issue comment |

- An item lives in the vault or in memory, never both. Memory points to a vault note instead of
  copying it.
- A memory entry the session read or wrote that holds system or project knowledge moves to the vault;
  the entry is then deleted, or reduced to a pointer to the note.
- A promise not kept is a follow-up. If it is knowledge (a runbook, a write-up), write it in the vault.

Issues are only those linked to the session: named in the conversation, or in a branch, commit, or PR
it touched. Never create an issue and never offer to; a follow-up with no issue goes to the task
index (step 5).

## 4. Ask everything in one batch

Before writing anything, send one message with:

1. **Unclear issue.** Each item that belongs to no issue or to more than one: which issue, or the task
   index's follow-ups list? Give a default.
2. **Leftover changes on a finished task** (uncommitted changes, unpushed commits, a worktree): clean
   up and forget, or keep for later? Leftovers on a task still in progress are work in progress: no
   question, step 5 records them.
3. **One draft per linked issue**, in full. A comment for teammates: what was done, PRs as links,
   decisions with their reasons, what is left; no vault links or paths, no agent internals. Propose
   closing only when every linked PR is merged and the acceptance criteria are met; otherwise a
   comment, with a state change if one fits.

Then stop and wait for the answers.

## 5. Write the vault and memory

Prefer the closest existing note and copy its siblings' conventions (folder, filename, frontmatter,
tags, links):

| Item | Target | How |
|---|---|---|
| Tied to an issue | The issue's note | Add a dated section (`## <Topic> YYYY-MM-DD`) |
| How a system works | Its reference note | Edit in place; correct what is now wrong |
| A procedure | Its runbook | Edit in place, or create one |
| No note fits | New note next to its closest siblings | Same frontmatter and naming as them |

Bump an `updated` field when the note has one, and link related notes. Do not commit, push, or pull in
the vault: the user's sync versions it.

If the vault has a task index (a note listing current work, such as `Ongoing Tasks.md`), keep it in
sync:

- Each issue's section: tracker state, a dated line for this session, the next action.
- Work in progress: its branch and worktree path.
- "Keep for later" leftovers and follow-ups with no issue: an entry in its follow-ups list. For a
  leftover, give repo, branch, worktree path, a one-line summary of the changes, and its issue.

Write memory in the agent's own memory format.

## 6. Post what was approved

Post a draft only after a yes given to that draft. A draft the answers changed is shown again and
needs a new yes. A close goes with its comment: when the comment waits, the close waits too.

For each issue actually closed: delete its section from the task index, set its note's status to
done, and move the note into the archive folder next to it, if one exists (such as `Attic/`).

**No tracker write without a yes to the exact text.** That covers comments, state changes, closing,
and creating issues.

| Thought | Reality |
|---|---|
| "The user is leaving, posting saves them time" | A post they did not read can't be taken back. Show the draft. |
| "It's only a status comment" | Teammates read it as the user's words. |
| "They said make sure nothing gets lost" | Nothing is lost: the draft is in the report. |
| "This follow-up deserves its own ticket" | Put it in the task index's follow-ups list. |

## 7. Confirm, then clean up

Answering "clean up" in step 4 is a choice, not the confirmation. For each leftover to clean up, show
what will be lost — `git -C <worktree> status --short`, `git -C <worktree> diff --stat`,
`git log --oneline <branch> --not --remotes` — and ask "Delete it?" at the end of the report. Delete
only after that yes:

```sh
git -C <repo> worktree remove --force <worktree>
git -C <repo> branch -D <branch>
```

Skip the confirmation only when the user's own words named that cleanup before you asked, such as
"close the session and delete the PLA-851 worktree".

## 8. Report

The final answer is these parts, in this order:

1. **Vault**: each note created or changed, with one line on what it now holds.
2. **Memory**: entries added, changed, moved to the vault, or removed.
3. **Tracker**: each action posted; each draft still waiting, in full.
4. **Loose ends**: how each was settled, or the "Delete it?" question from step 7.
5. **Re-routed or skipped**: anything that could not reach its home, and where it went instead.
