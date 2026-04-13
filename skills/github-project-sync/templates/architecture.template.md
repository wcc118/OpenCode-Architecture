# Architecture — {{PROJECT_NAME}}

> **Maintained by:** Orchestrator agent  
> **Last updated:** {{TIMESTAMP}}  
> **Session:** {{SESSION_ID}}

---

## Project Overview

**Purpose:** {{PROJECT_PURPOSE}}  
**Primary language(s):** {{LANGUAGES}}  
**Runtime/framework:** {{FRAMEWORK}}  
**Repo:** {{GITHUB_REPO_URL}}  
**Branch strategy:** `main` (protected) → `develop` → `feature/*` / `fix/*` / `hotfix/*`

---

## Repository Structure

```
{{REPO_ROOT}}/
├── {{STRUCTURE_PLACEHOLDER}}
```

> Orchestrator updates this tree after any structural change (new dir, deleted module, renamed file).

---

## Key Modules

| Module / File | Purpose | Owner Agent | Last Modified |
|---|---|---|---|
| {{MODULE}} | {{PURPOSE}} | {{AGENT}} | {{DATE}} |

---

## Data Flow

```
{{DATA_FLOW_DIAGRAM}}
```

---

## External Dependencies

| Package | Version | Purpose | Audit Status |
|---|---|---|---|
| {{PACKAGE}} | {{VERSION}} | {{PURPOSE}} | ✅ / ⚠️ / ❌ |

> `dependency-audit` skill updates this table before any install or version change.

---

## Agent Responsibilities in This Project

| Agent | Role in This Project | Allowed Actions |
|---|---|---|
| orchestrator | Session lifecycle, git ops, routing | Full |
| build | Code generation, file editing | Full |
| analysis | Planning, code review | Read-only |
| refactor | Code cleanup, quality | Standard |
| test | Test generation and runs | Full test tools |
| debug | Error analysis | Read-only |
| docs | Documentation | Read-only |
| explore | Codebase navigation | Read-only |
| vision | Image/diagram analysis | Read-only, no bash |

---

## Known Constraints & Decisions

| Decision | Rationale | Date | Session |
|---|---|---|---|
| {{DECISION}} | {{RATIONALE}} | {{DATE}} | {{SESSION_ID}} |

---

## Change Log

| Date | Session | Agent | Change Summary |
|---|---|---|---|
| {{DATE}} | {{SESSION_ID}} | {{AGENT}} | {{SUMMARY}} |
