---
name: codebase-orientation
description: Orients any agent to the current project state before taking any action. ALWAYS invoke at the start of every agent session, before editing any file, before running any command, or any time an agent seems lost or out of context. Global skill — applies to all agents at all tiers.
---

# Codebase Orientation

Every agent runs this before doing anything else. No exceptions.

## Why This Exists

Agents lose context. Sessions restart. Branches diverge. Without orientation, an agent will confidently edit the wrong file, work on a cancelled task, or miss that the architecture changed in the last session. This skill is the fix.

---

## Orientation Sequence

Run these steps in order. Do not skip any.

### 1. Read the State Machine
```
READ agent.md
```
Extract and hold in active context:
- Current **Phase**
- **Active branch**
- **Last known good commit** SHA
- **Next Required Action** (bottom of file)
- Any **Active Errors**
- **Hook Status** table — which hooks are already done this session

### 2. Read the Architecture Map
```
READ architecture.md
```
Extract and hold in active context:
- Project purpose and language/framework
- Repository structure (top-level dirs and their roles)
- Key modules relevant to the current task
- Any constraints or decisions relevant to current work

### 3. Confirm Branch
```bash
git branch --show-current
git status
```
- If branch does not match `agent.md` active branch → STOP, report to orchestrator
- If there are uncommitted changes from a previous session → STOP, report to orchestrator
- If clean and correct → proceed

### 4. Confirm Task Context
From `agent.md`, confirm you understand:
- What task you are working on
- What the acceptance criteria are
- Which hooks you still need to complete before committing

### 5. Report Orientation Summary

Before proceeding with any work, output a brief orientation summary:

```
📍 ORIENTATION COMPLETE
Project: [name]
Phase: [phase]
Branch: [branch]
Task: [current task]
Remaining hooks: [list any uncompleted hooks]
Next action: [what I am about to do]
```

---

## Re-Orientation Triggers

Run orientation again (from Step 1) if any of these occur mid-session:
- You receive an error you didn't expect
- You are about to touch a file outside your expected scope
- You've completed a subtask and are starting the next one
- You notice `agent.md` has been updated since you last read it
- Your context feels stale or uncertain

## For Subagents

Subagents (`debug`, `docs`, `explore`, `vision`) run a lighter orientation:
- Read `agent.md` → extract Phase, active branch, current task only
- Read the specific section of `architecture.md` relevant to their task
- Confirm branch
- Report what they are about to do

Subagents do NOT need to check hook status — that is the orchestrator's responsibility.
