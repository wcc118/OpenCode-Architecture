---
name: session-memory
description: Writes a structured session log at the end of every agent session. Invoke before any commit, at session end, or whenever a significant milestone is reached. Global skill — all primary agents and the orchestrator write session logs. These logs form the recursive learning corpus for model improvement.
---

# Session Memory

Every session that touches code ends with a session log. This is non-negotiable.

The log serves three purposes:
1. **Continuity** — next session picks up exactly where this one left off
2. **Recursive learning** — logs are the training corpus for improving model performance
3. **Git gate** — the pre-commit hook blocks commits until a log exists

---

## Session Log Format

Write to: `sessions/session_YYYYMMDD_HHMMSS.md`

Use the current timestamp at the moment of writing, not the session start time.

```markdown
# Session Log — {{TIMESTAMP}}

**Project:** {{PROJECT_NAME}}  
**Agent:** {{AGENT_NAME}}  
**Branch:** {{BRANCH}}  
**Session ID:** session_{{YYYYMMDD_HHMMSS}}  
**Duration:** ~{{N}} minutes  

---

## Goal

{{One paragraph: what this session set out to accomplish}}

---

## What Was Done

{{Ordered list of concrete actions taken. Be specific — file names, function names, commands run.}}

1. Created `src/auth/jwt.py` with `generate_token()` and `verify_token()` functions
2. Added `python-jose` dependency, passed dependency-audit
3. Wrote 8 unit tests in `tests/test_auth.py` — all passing
4. Updated `architecture.md` Key Modules table with auth module entry

---

## Decisions Made

{{Decisions that future agents need to know about.}}

| Decision | Rationale | Alternatives Considered |
|---|---|---|
| Used HS256 for JWT signing | Simpler for single-service setup | RS256 (rejected: adds key management overhead) |

---

## What Worked

{{Techniques, approaches, or patterns that produced good results. Be specific enough that a model could learn from this.}}

- Breaking the auth flow into pure functions first, then wiring to routes, made testing much easier
- Reading architecture.md before starting caught that the existing middleware pattern expected async functions

---

## What Failed / Blockers

{{Errors encountered, dead ends, things that didn't work. Include error messages if short enough.}}

| Problem | Root Cause | Resolution |
|---|---|---|
| `ImportError: jose not found` | Dependency not installed | Ran `pip install python-jose`, added to requirements.txt |
| Test mock for `datetime.utcnow` not working | Wrong patch target | Changed patch path to `src.auth.jwt.datetime` |

---

## Skill Improvement Notes

{{Observations about how agent skills or prompts could be improved. This section feeds directly into skill iteration.}}

- `error-debug-loop` correctly identified the wrong patch target on first attempt — works well for import/mock errors
- `codebase-orientation` orientation summary could include the test directory path — had to search for it manually

---

## Open Items

{{What is NOT done. What the next session needs to pick up.}}

- [ ] Wire auth functions to `/api/login` and `/api/refresh` routes (branch: `feature/auth-routes`)
- [ ] Add integration tests for token expiry edge case
- [ ] Update API section of `architecture.md` once routes are wired

---

## State Handoff

{{Fill in exactly — orchestrator uses this to update agent.md}}

**Next required action:** {{NEXT_ACTION}}  
**Branch to continue on:** {{BRANCH}}  
**Last commit SHA:** {{SHA}}  
**Agent.md Phase should be set to:** {{PHASE}}  
```

---

## Writing Guidelines

**Be concrete, not vague.**
- ❌ "Fixed some auth issues"
- ✅ "Fixed `verify_token()` raising `KeyError` when `exp` claim missing — added `.get()` with default"

**Skill improvement notes are the most valuable section.** Write them even if small. They compound.

**Open items must be actionable.** Each one should be completable in a single future session.

**The State Handoff section is mandatory.** Orchestrator reads it to update `agent.md` after you write the log.

---

## After Writing the Log

1. Confirm the file is saved to `sessions/`
2. Notify the orchestrator: "Session log written: `sessions/session_{{ID}}.md`"
3. Orchestrator updates `agent.md` using the State Handoff section
4. Commit can now proceed
