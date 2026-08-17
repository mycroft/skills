---
name: editing-skills
description: Use when changing, fixing, or extending a skill that already exists — including editing its description or supporting files, or responding to a skill that failed to trigger or was followed incorrectly
---

# Editing Skills

## Overview

Editing a live skill is riskier than writing a new one: agents already rely on its current wording, and a regression is silent. **The Iron Law applies to edits — no skill change without a failing test first.**

**REQUIRED SUB-SKILL:** You MUST use superpowers:writing-skills before editing. It owns the frontmatter rules, the Match the Form to the Failure table, and the subagent testing methodology. This skill covers only what is different about *changing* a skill that already ships.

## Diagnose Before You Edit

Two failures both get reported as "the skill didn't work", and they have different fix sites:

| Symptom | Failure | Fix site |
|---|---|---|
| Agent never loaded the skill | Discovery | frontmatter `description` — triggers and symptoms only |
| Agent loaded it and did the wrong thing | Compliance or output shape | body |

Check the transcript for whether the skill was actually invoked. Guessing wrong means adding body text that nobody ever reads.

## The Loop

1. **Read the whole skill** — SKILL.md and every supporting file. Restating guidance that is already there is the most common defect in skill edits.
2. **RED — reproduce against the current skill.** One fresh subagent, skill present, the task that failed. Capture the rationalization or wrong output verbatim. *If it passes, there is nothing to fix — stop, and drop the planned edit.*
3. **Pin the regression set.** Write down what the skill already gets right, and pick 2-3 scenarios covering it. This is what your edit must not break.
4. **Edit minimally**, matching the form to the failure you just observed. Prefer *replacing* wording over appending to it.
5. **GREEN — re-run the new scenario AND the regression set.** Both must pass. A fix that breaks a case the skill used to handle is not a fix.
6. **Report** the diff, the baseline failure verbatim, and the results of both runs.

## Budget the Edit

Skills die by accretion: every edit adds a paragraph, and eventually nothing in the document is binding.

- Run `wc -w SKILL.md` before and after. Growth needs a justification.
- Over ~500 words, your edit should make the skill *shorter*: move heavy reference into a supporting file, cut what your new wording supersedes.
- Delete guidance your change makes obsolete. Two rules on the same subject are a loophole.

## Never Append a Nuance Clause

"...unless it matters", "...but use judgment", "(this doesn't apply to X)" — these reopen the negotiation and measurably degrade guidance that previously worked. Express a real exception as its own conditional keyed to an observable predicate, or restructure so the rule cannot reach the part that must be exempt.

## Red Flags - STOP

- "I can see what's wrong, I don't need a baseline" — then you have not shown the current skill fails.
- "I'm only adding a section, that isn't really a change" — it is. Test it.
- "I'll append this now and tidy up later" — nobody tidies up later.
- "The existing scenarios obviously still pass" — run them.
- Editing the description to summarize the workflow — that builds a shortcut agents take instead of reading the skill.

**All of these mean: run RED first.**
