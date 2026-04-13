---
name: rollback-recovery
description: Structured rollback and recovery protocol for when builds break, tests fail catastrophically, or a branch reaches an unrecoverable state. Invoke immediately when build is bricked, all tests fail after a change, or the orchestrator detects a change has corrupted project state. Orchestrator-scoped.
---

# Rollback Recovery

When something bricks the build, the priority is: **protect main, restore a working state, understand what happened, fix it safely.**

Never attempt to fix a bricked state on the same branch through blind edits. Roll back first, then fix forward on a new branch.

---

## Trigger Conditions

Invoke this skill when:
- Build fails completely after a commit (was passing before)
- All tests fail after a merge or large change
- Application crashes on startup after a change
- A merge produced conflicts that corrupted files
- `agent.md` shows 3 consecutive failed attempts on same error

---

## Recovery Protocol

### Step 1 — Stop Everything

```
ROLLBACK INITIATED
==================
Timestamp:    {{ISO timestamp}}
Trigger:      {{what caused the rollback decision}}
Agent:        {{which agent was working}}
Branch:       {{current branch}}
Last good SHA: {{SHA from agent.md}}
```

Set `agent.md` Phase to `ROLLBACK`. No other agent takes any action until Phase is cleared.

### Step 2 — Preserve the Evidence

Before touching any git state, capture what went wrong:

```bash
# Save current state for post-mortem
git diff HEAD > /tmp/rollback_{{timestamp}}_diff.patch
git log --oneline -10 > /tmp/rollback_{{timestamp}}_log.txt
{{test command}} 2>&1 > /tmp/rollback_{{timestamp}}_failures.txt
```

This evidence goes into the session log and helps prevent the same issue recurring.

### Step 3 — Assess Rollback Scope

| Situation | Rollback Strategy |
|---|---|
| Uncommitted changes broke things | Stash or discard working tree changes |
| Last 1-2 commits broke things | Soft reset or revert those commits |
| Feature branch is unrecoverable | Abandon branch, start fresh from develop |
| Develop branch is corrupted | Restore develop from last known good tag/commit |
| Main branch affected | **STOP. Alert human immediately. Do not touch main.** |

### Step 4 — Execute Rollback

#### Option A: Uncommitted changes
```bash
git stash push -m "rollback-{{timestamp}}: pre-rollback stash"
# verify build/tests pass
# if yes: branch is recovered, analyze stash for what broke it
# if no: something is wrong beyond the working tree changes
```

#### Option B: Revert recent commits (safe for shared branches)
```bash
git revert {{bad commit SHA}}..HEAD --no-commit
git commit -m "revert: rollback to last known good state {{good SHA}}"
git push origin {{branch}}
```

#### Option C: Reset to last known good (unshared branch only)
```bash
git checkout {{branch}}
git reset --hard {{last good SHA from agent.md}}
git push --force-with-lease origin {{branch}}
# ONLY if this branch has not been merged anywhere
```

#### Option D: Abandon branch, restart from develop
```bash
git checkout develop
git pull origin develop
git checkout -b fix/{{original-feature}}-v2
# note: v2 signals a restart, helps with session continuity
```

### Step 5 — Verify Recovery

After rollback:
```bash
{{build command}}          # must succeed
{{test command}}           # must pass (or return to pre-regression state)
{{startup command}}        # application must start
```

If verification fails: escalate to human. Do not attempt further automated recovery.

### Step 6 — Open Fix Branch

Once a working state is restored, create a new branch to fix the underlying problem:

```bash
git checkout -b fix/{{description-of-what-broke}}
```

Before writing any code on this branch, route to `@debug` subagent with the preserved evidence files to diagnose root cause. Do not guess.

### Step 7 — Update Agent.md and Write Session Log

Update `agent.md`:
- Phase: `BUILDING` (or `PLANNING` if root cause unclear)
- Add rollback event to error table with full summary
- Update last known good SHA
- Set Next Required Action to the fix branch task

Write session log immediately — even if the session is not "done." A rollback event is always worth logging.

---

## Post-Mortem Template

Add this block to the session log's "What Failed" section:

```markdown
### Rollback Event

**Trigger:** {{what caused the rollback}}
**Branch affected:** {{branch}}
**Commits rolled back:** {{SHAs}}
**Root cause (if known):** {{diagnosis}}
**Root cause (if unknown):** Under investigation on fix/{{branch}}
**Evidence preserved at:** /tmp/rollback_{{timestamp}}_*
**Recovery strategy used:** Option {{A/B/C/D}}
**Recovery verified:** ✅ / ❌

**Prevention:** {{what would have caught this earlier}}
```

---

## What Rollback-Recovery Does NOT Do

- It does not diagnose the root cause — that is `error-debug-loop`'s job
- It does not touch `main` under any circumstance
- It does not delete session logs or git history
- It does not retry the original approach — a rollback means the approach needs rethinking
