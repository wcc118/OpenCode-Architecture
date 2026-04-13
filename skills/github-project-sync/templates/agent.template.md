# Agent State — {{PROJECT_NAME}}

> **This file is the single source of truth for current session state.**  
> Every agent MUST read this file before taking any action.  
> Orchestrator MUST update this file after every meaningful state change.  
> Last updated: {{TIMESTAMP}} | Session: {{SESSION_ID}}

---

## 🔴 Current Status

**Phase:** `{{PHASE}}`  
<!-- Phases: INIT | PLANNING | BUILDING | TESTING | REVIEWING | COMMITTING | COMPLETE | BLOCKED | ROLLBACK -->

**Active agent:** `{{ACTIVE_AGENT}}`  
**Active branch:** `{{BRANCH}}`  
**Last known good commit:** `{{LAST_GREEN_SHA}}`

---

## 📋 Current Task

**Goal:** {{TASK_GOAL}}  
**Assigned to:** {{AGENT}}  
**Started:** {{TIMESTAMP}}  
**Subtask:** {{CURRENT_SUBTASK}}

### Acceptance Criteria
- [ ] {{CRITERION_1}}
- [ ] {{CRITERION_2}}

---

## 🪝 Hook Status (This Session)

These hooks are MANDATORY. Orchestrator checks completion before advancing phase.

| Hook | Required At | Status | Completed |
|---|---|---|---|
| Read `architecture.md` | Session start | ⬜ / ✅ | — |
| Read `agent.md` | Session start | ⬜ / ✅ | — |
| Branch created/confirmed | Before any edit | ⬜ / ✅ | — |
| `dependency-audit` | Before any install | ⬜ / N/A | — |
| Tests pass | Before commit | ⬜ / ✅ / N/A | — |
| Session log written | Before commit | ⬜ / ✅ | — |
| Commit pushed | Session end | ⬜ / ✅ | — |
| `architecture.md` updated | If structure changed | ⬜ / ✅ / N/A | — |

---

## 📚 Task Queue

### In Progress
1. {{TASK}} — `{{AGENT}}` — branch: `{{BRANCH}}`

### Pending
1. {{TASK}}
2. {{TASK}}

### Completed This Session
1. ✅ {{TASK}} — commit: `{{SHA}}`

### Blocked
- ⛔ {{TASK}} — Reason: {{REASON}}

---

## 🐛 Active Errors

| Error | Agent | Attempts | Status |
|---|---|---|---|
| {{ERROR_SUMMARY}} | {{AGENT}} | {{N}} | 🔴 Open / 🟡 In Progress / ✅ Resolved |

> `error-debug-loop` skill owns this table. Updates after every error event.

---

## 🌿 Branch State

| Branch | Purpose | Status | Last Commit |
|---|---|---|---|
| `main` | Protected, production | 🟢 Clean | {{SHA}} |
| `develop` | Integration | 🟢 / 🟡 | {{SHA}} |
| `{{FEATURE_BRANCH}}` | {{PURPOSE}} | 🔵 Active | {{SHA}} |

---

## 🔁 Next Required Action

> **ORCHESTRATOR: Execute this before anything else.**

```
{{NEXT_ACTION}}
```

<!-- Examples:
  - READ architecture.md then confirm branch feature/auth-flow exists
  - RUN test suite on branch feature/parser before committing
  - WRITE session log then merge develop → main
  - INVOKE rollback-recovery: tests failed on commit abc123
-->

---

## 📎 Context Pointers

Quick references so any agent can orient instantly:

- **Entry point:** `{{ENTRY_FILE}}`
- **Config:** `{{CONFIG_FILE}}`
- **Tests:** `{{TEST_DIR}}`
- **Docs:** `{{DOCS_DIR}}`
- **Sessions:** `sessions/`
- **Last session log:** `sessions/{{LAST_SESSION_FILE}}`
