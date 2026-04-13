---
name: task-decomposer
description: Breaks any incoming goal into atomic, verifiable subtasks before any agent begins work. Invoke at the start of every new task or feature request, before routing to any agent. Orchestrator-scoped. Prevents agents from attempting tasks too large for a single session and ensures every subtask has clear acceptance criteria.
---

# Task Decomposer

No agent starts work until the task is decomposed. Decomposition is the orchestrator's first move on every new goal.

---

## Why Decompose First

Large tasks cause agents to:
- Lose context mid-execution and make inconsistent decisions
- Produce work that can't be tested in isolation
- Create commits that are too large to safely review or roll back
- Drift from the original goal as complexity compounds

A well-decomposed task gives every agent a clear, bounded, verifiable unit of work.

---

## Decomposition Protocol

### Step 1 — Understand the Goal

Before decomposing, answer these questions:

1. What does "done" look like? (end state, not process)
2. What must NOT change? (constraints, protected code, locked dependencies)
3. Are there external dependencies? (API, DB schema, other teams' code)
4. What is the failure mode if this goes wrong?

If any answer is unclear, ask the human before decomposing.

### Step 2 — Identify Natural Boundaries

Good subtask boundaries are:
- **Testable in isolation** — you can write a test that passes/fails for just this piece
- **Single responsibility** — one agent, one concern, one branch
- **Reversible** — if it fails, rolling it back doesn't break other subtasks
- **Small enough for one session** — rule of thumb: completable in under 60 minutes

Bad boundaries (split these further):
- "Build the auth system" — too large
- "Fix the bug" — too vague
- "Update everything to use the new API" — too broad

### Step 3 — Write the Subtask List

For each subtask, define:

```markdown
## Subtask {{N}}: {{Short Name}}

**Goal:** {{One sentence — what this subtask produces}}
**Agent:** {{which agent does this work}}
**Branch:** feature/{{slug}} or fix/{{slug}}
**Depends on:** Subtask {{N-1}} complete / none
**Estimated sessions:** {{1-3}}

### Acceptance Criteria
- [ ] {{Specific, verifiable criterion}}
- [ ] {{Specific, verifiable criterion}}
- [ ] Tests written and passing for this subtask
- [ ] Session log written

### Out of Scope
- {{What this subtask explicitly does NOT do}}
```

### Step 4 — Sequence the Subtasks

Identify dependencies and order them:

```
[Subtask 1] → [Subtask 2] → [Subtask 4]
                           ↗
              [Subtask 3] →
```

Subtasks with no dependencies can run in parallel (different branches). Subtasks that depend on each other must run in sequence.

### Step 5 — Write to Agent.md

Add all subtasks to the `agent.md` Task Queue in order:

```markdown
### Pending
1. {{Subtask 1 name}} — `@build` — branch: `feature/{{slug}}`
2. {{Subtask 2 name}} — `@refactor` — branch: `feature/{{slug}}`  
3. {{Subtask 3 name}} — `@test` — branch: `feature/{{slug}}`
```

Set the first subtask to "In Progress" and update the Next Required Action.

---

## Decomposition Examples

### Example: "Add user authentication"

❌ Bad: One task "implement auth"

✅ Good decomposition:
1. `feat/auth-models` — Define User model and DB schema (@build)
2. `feat/auth-jwt` — Implement JWT generate/verify functions (@build)
3. `feat/auth-routes` — Wire /login and /refresh endpoints (@build)
4. `test/auth-unit` — Unit tests for JWT functions (@test)
5. `test/auth-integration` — Integration tests for routes (@test)
6. `docs/auth` — Update architecture.md and API docs (@docs)

### Example: "Fix the broken search"

❌ Bad: One task "fix search"

✅ Good decomposition:
1. `explore/search-diagnosis` — Map current search flow, identify failure point (@explore → report to orchestrator)
2. `fix/search-query` — Fix identified root cause (@build)
3. `test/search-regression` — Add regression test for this specific failure (@test)

---

## Subtask Sizing Guide

| Subtask feels like... | Action |
|---|---|
| More than one agent needs to be involved | Split it |
| You can't write a single acceptance criterion | Split it |
| It touches more than 3 files | Consider splitting |
| It could be done in one focused session | Good size |
| It's a single function or endpoint | Perfect size |
| It's just a config change or doc update | Fine as-is |
