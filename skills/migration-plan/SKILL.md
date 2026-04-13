---
name: migration-plan
description: Prepare a structured migration plan with intro, steps, verification, and revert paths.
---

When asked to write a migration plan, gather enough context first, then produce a document following the structure below.

## Phase 1 — Gather Context

Before writing anything, make sure you know:

- **What** is being migrated (database schema, infrastructure, service, data, config, etc.)
- **Why** the migration is needed (motivations, constraints, deadlines)
- **Who** is affected (teams, services, end-users)
- **Risk level**: estimated blast radius if something goes wrong

If any of these are unclear, ask the user before proceeding.

## Phase 2 — Output Structure

Produce the plan using the following template:

---

```
# Migration Plan: <short title>

## Introduction

[2–4 sentences describing what is being migrated and why.
Include the motivation — technical debt, compliance, performance, new architecture, etc.
Mention any known constraints or non-negotiable requirements.]

## Scope

- **Systems affected**: [list services, databases, infra components]
- **Teams involved**: [owners, reviewers, on-call]
- **Estimated duration**: [per step and total] *(initial estimate; update after dry-run)*
- **Risk level**: Low / Medium / High

---

## Steps

### Step N — <title>

**Window**: in-migration / out-migration / maintenance window / rolling
**Owner**: [team or person responsible]

#### What

[1–3 sentences describing what this step does.]

#### How

1. [Concrete action]
2. [Concrete action]
3. ...

#### Verification

- [ ] [Observable signal that confirms the step succeeded — metric, log line, query result, health check, etc.]
- [ ] [Secondary check if applicable]

#### Revert Path

> If this step needs to be rolled back:

1. [Step to undo the change]
2. [Step to restore prior state]
3. [Signal that confirms rollback is complete]

---

[Repeat for each step]

---

## Dry-Run / Rehearsal

> Include this section when risk level is **Medium or High**.

Describe a rehearsal that can be executed before the real migration — ideally on a staging environment or against a non-production dataset.

- **Goal**: validate assumptions, surface surprises, and time each step
- **Checklist**:
  - [ ] Rehearsal executed on staging / shadow environment
  - [ ] Step durations recorded (use them to set realistic maintenance window estimates)
  - [ ] Revert path tested end-to-end at least once
  - [ ] Any deviation from the plan documented and plan updated accordingly

> If a full rehearsal is not possible, list what was validated and what remains untested.

---

## Communication

- **Notify before**: [who to inform, how far in advance, and via what channel]
- **Notify during**: [who is on standby; escalation path if something goes wrong]
- **Notify after**: [who to confirm completion to, and what the message should cover]

---

## Post-Migration Checklist

- [ ] All verification checks passed
- [ ] Monitoring shows no anomalies
- [ ] Old infrastructure / data / config decommissioned (if applicable)
- [ ] Runbook updated
- [ ] Stakeholders notified

## Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| [e.g. data loss during copy] | Low | High | [dry-run + checksums before cutover] |

```

---

## Guidelines

- Keep step titles short and verb-first: "Enable feature flag", "Backfill legacy rows", "Cut over DNS".
- Separate **in-migration** steps (can run alongside current system) from **out-migration** steps (require a maintenance window or traffic pause) — label each step clearly.
- Verification must be observable: a command to run, a dashboard to check, a query to execute — not "make sure it works".
- Revert paths must be actionable: assume the person executing them is under pressure and has no context beyond what is written.
- Flag any step that is **irreversible** (e.g. destructive schema change, data deletion) with a `> ⚠ Irreversible` callout so it stands out.
- If the migration has dependencies between steps, call them out explicitly: "Step 3 requires Step 2 to have been running for at least 24h."
