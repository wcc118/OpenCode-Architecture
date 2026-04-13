---
name: context-window-manager
description: Monitors and manages context window consumption during long agent sessions. Invoke proactively at 40% consumption — do not wait for problems. Global skill — all agents use this. Contains specific recovery steps for compaction loops where OpenCode's built-in compaction runs repeatedly without making progress.
---

# Context Window Manager

Context overflow is the most common failure mode in long agent sessions. The fix is proactive compression at 40% — not reactive recovery at 80%. By the time OpenCode's built-in compaction triggers, there is often not enough headroom left to compress meaningfully, causing a compaction loop that makes no progress.

This skill breaks that loop and prevents it from starting.

---

## Trigger Thresholds

### Proactive triggers — invoke before problems start:
- You have completed 2 or more subtasks in a single session
- You have read more than 5 files in a single session
- You estimate you are at or past **40%** of your context window
- A session has been running for more than ~30 minutes of active work
- You are about to route to a subagent and context is already substantial

### Reactive triggers — something is already wrong:
- You are unsure what was decided earlier in the session
- You catch yourself re-reading a file you already processed
- Your responses are becoming less consistent with earlier work
- OpenCode's compaction has run more than once in the same session
- You feel like you are spinning without making progress

### Compaction loop signal — stop immediately if you see this:
- OpenCode compaction has run AND context filled again quickly AND compaction ran again
- You are producing summaries of summaries
- The same content keeps reappearing in your context after compaction

---

## Protocol by Consumption Level

### Level 1 — Low (under 40%)
No action needed. Make a mental note of the check and continue.

Prevention habits to maintain at this level:
- Reference files by name only once you have extracted what you need
- Release subagent outputs immediately after extracting the key finding
- Do not hold completed subtask details in active context

### Level 2 — Medium (40–60%) — invoke this skill now
Light compression. Do all of these before continuing:

1. Write any in-progress session log content to disk immediately — do not hold it in memory
2. Summarize completed subtasks to one line each, release the detail
3. Drop all full file contents from active consideration — filenames only from here
4. Verify your task queue is stored in `agent.md`, not held in context
5. Continue working — no session break needed yet

```
COMPRESSION NOTE — MEDIUM
==========================
Completed: {{subtask}} → {{one-line outcome}}
Completed: {{subtask}} → {{one-line outcome}}
Active: {{current subtask}} on {{branch}}
Remaining queue: see agent.md
Context released: {{list of files/outputs dropped}}
```

### Level 3 — High (60–75%) — active compression required
Write a checkpoint and re-anchor before continuing.

1. Write checkpoint to `sessions/checkpoint_YYYYMMDD_HHMMSS.md`:

```markdown
# Checkpoint — {{TIMESTAMP}}

**Subtask completed:** {{name}} (or in-progress: {{name}})
**Branch:** {{branch}}
**Last commit SHA:** {{SHA}}

## Remaining task queue (names only)
- {{name}}
- {{name}}

## Decisions affecting remaining work
- {{one line}}

## Next immediate action
{{single sentence — this is your re-entry point}}
```

2. Update `agent.md` Next Required Action to match the checkpoint
3. Release everything — task details, file contents, subagent outputs, error traces
4. Re-read `agent.md` only as your fresh context anchor
5. Do not re-read `architecture.md` unless the next subtask specifically requires it
6. Continue from the checkpoint's "Next immediate action"

### Level 4 — Critical (75%+) — hard stop, fresh session required
Do not attempt to continue. Save state and hand off cleanly.

1. Write checkpoint (as Level 3 above)
2. Write partial session log to `sessions/session_YYYYMMDD_HHMMSS.md` — mark title `[PARTIAL]`

Session log must include at minimum:
- What was completed this session (one line per subtask)
- What is in progress and exactly where it stopped
- Branch name and last commit SHA
- Any open errors or blockers
- The single next action for the fresh session to take

3. Update `agent.md`:
   - Phase: `HANDOFF`
   - Next Required Action: the exact first thing the fresh session should do
   - Last known good SHA: current HEAD

4. Commit any completed work that has not been committed yet
5. Push current branch
6. Tell the human clearly:

```
Context at critical level. State has been saved:
- Checkpoint: sessions/checkpoint_{{ID}}.md
- Session log: sessions/session_{{ID}}.md (PARTIAL)
- agent.md updated with next action

Please start a fresh OpenCode session. The orchestrator will
read agent.md and resume exactly where we left off.
```

---

## Compaction Loop Recovery

If OpenCode's built-in compaction has already triggered and is looping without making progress, follow these steps immediately. Do not attempt further compaction or summarization.

### Step 1 — Recognize the loop
Signs you are in a compaction loop:
- Compaction ran, context refilled quickly, compaction ran again
- You are generating summaries of already-summarized content
- No meaningful work is completing between compaction events

### Step 2 — Hard stop
Stop all agent routing and task work immediately.

### Step 3 — Minimal state capture
Write only what is essential — keep this short or compaction will loop again:

```markdown
# Compaction Recovery — {{TIMESTAMP}}

Branch: {{branch}}
Last SHA: {{output of: git rev-parse HEAD}}
Phase: COMPACTION_RECOVERY

What was in progress: {{one sentence}}
Next action when fresh: {{one sentence}}
```

Save to `sessions/checkpoint_YYYYMMDD_HHMMSS_compaction_recovery.md`

### Step 4 — Commit anything committed-able
```bash
git status                          # what is staged or modified?
git add {{any completed work}}
git commit -m "wip: compaction recovery checkpoint — {{brief description}}"
git push origin {{branch}}
```

### Step 5 — Update agent.md minimally
Write only the Next Required Action field. Do not re-read the whole file.

### Step 6 — Tell the human
```
Compaction loop detected and stopped. Minimal state saved to sessions/.
agent.md Next Required Action updated.
Please start a fresh session to continue.
```

---

## Re-Anchoring After Any Compression

After compression at Level 2 or above, confirm these four things before proceeding:

```
RE-ANCHOR CHECK
===============
1. What is my current active task? (from agent.md)
2. What branch am I on? (confirm with git branch --show-current)
3. What are the acceptance criteria for this task?
4. What is my single next action?
```

Only proceed when all four are clear. If any is unclear, read `agent.md` again — do not guess.

---

## For the Orchestrator — Monitoring Subagents

Watch for these signals that a subagent is context-compressed:
- Returns an unusually short or vague response
- Contradicts a decision it made earlier in the session
- Asks a question it already answered
- Returns a summary of a summary

When you see these signals:
1. Do not send more work to that subagent
2. End its session
3. Write its partial output to disk if anything useful was produced
4. Start a fresh subagent invocation with a clean, minimal prompt re-injected from `agent.md`

---

## Prevention — The Habits That Prevent All of This

These habits, maintained consistently, prevent context overflow from ever becoming critical:

- Write session logs and checkpoints incrementally after every subtask — never hold them in memory until the end
- Reference files by name only after reading them — never hold full contents
- Enforce the 200-word subagent response limit on every single invocation
- Keep subtasks small (per `task-decomposer`) so each one completes before context pressure builds
- Compress at 40% — every time, without exception
- Never re-read `architecture.md` between subtasks unless the next task specifically requires it
