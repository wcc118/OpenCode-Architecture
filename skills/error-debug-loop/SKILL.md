---
name: error-debug-loop
description: Structured error resolution protocol. Invoke immediately on any non-zero exit code, failed test, unhandled exception, or unexpected output. Global skill — all agents invoke this before retrying any failed operation. Prevents blind retry loops and builds a searchable error history.
---

# Error Debug Loop

When something breaks, stop. Do not retry blindly. Run this protocol.

---

## Trigger Conditions

Invoke this skill when:
- Any command exits with a non-zero code
- A test fails
- An unhandled exception is raised
- Output is clearly wrong (empty when it shouldn't be, wrong type, wrong shape)
- An agent has retried the same operation twice without success

---

## Protocol

### Step 1 — Capture

Record the full error context before doing anything else:

```
ERROR CAPTURE
=============
Timestamp:    {{ISO timestamp}}
Agent:        {{agent name}}
Branch:       {{current branch}}
Command/Action: {{exactly what was run}}
Exit code:    {{code}}
Error output:
---
{{full stderr / exception / traceback}}
---
Expected:     {{what should have happened}}
Actual:       {{what happened instead}}
```

### Step 2 — Classify

Classify the error into one of these categories. Classification determines where to look first.

| Class | Indicators | Look First At |
|---|---|---|
| **Syntax** | SyntaxError, unexpected token, parse error | The file that was just edited |
| **Import/Dependency** | ModuleNotFoundError, ImportError, cannot find module | requirements.txt / package.json, venv activation |
| **Runtime** | AttributeError, TypeError, KeyError, NullPointerException | The specific line in traceback |
| **Logic** | Wrong output, test assertion fails, unexpected value | The function's logic, edge cases, test fixture data |
| **Environment** | Works locally but not in test, permission denied, port in use | .env, system config, process conflicts |
| **Git** | Merge conflict, detached HEAD, rejected push | `git status`, `git log --oneline -5` |
| **Test** | Test itself is wrong, wrong mock, wrong fixture | The test file, not the source |

### Step 3 — Search Session History

Before attempting any fix, search previous sessions for this error pattern:

```bash
grep -r "{{key error term}}" sessions/ --include="*.md" -l
```

If a match is found, read that session's "What Failed" section. Apply the documented resolution if applicable. If it works, note it as "resolved via session {{ID}}" in your log.

### Step 4 — Diagnose

Based on classification, run targeted diagnostics. Do not run everything — run what the class points to.

**Syntax:**
```bash
python -m py_compile {{file}}        # Python
node --check {{file}}                # Node.js
{{linter}} {{file}}                  # language-specific linter
```

**Import/Dependency:**
```bash
pip list | grep {{package}}          # Python
cat requirements.txt | grep {{pkg}}
which python && python --version
```

**Runtime — read the traceback line by line:**
- Innermost frame is usually the actual error
- Work outward to find the caller that passed bad data

**Environment:**
```bash
env | grep {{relevant var}}
cat .env
ls -la {{expected path}}
```

### Step 5 — Attempt Fix

Apply the most targeted fix possible. Change the minimum amount of code to resolve the error.

Log the attempted fix:
```
ATTEMPT {{N}}
=============
Hypothesis:   {{what I think is wrong}}
Fix applied:  {{exactly what was changed}}
Result:       {{pass / fail / different error}}
```

### Step 6 — Verify

After fix is applied:
- Re-run the exact command that failed
- Run the full test suite for the affected module
- Confirm no new errors introduced

### Step 7 — Escalation Rules

| Attempts | Action |
|---|---|
| 1 | Apply fix, verify, continue |
| 2 | Try a different hypothesis, log both attempts |
| 3 | STOP. Update `agent.md` error table. Set Phase to BLOCKED. Report to orchestrator with full capture log. |

The orchestrator decides whether to route to `@debug` subagent, try a different approach, or escalate to the human.

---

## Updating Agent.md Error Table

After any error event (open or resolved), update the error table in `agent.md`:

```markdown
| TypeError in verify_token() | build | 2 | 🟡 In Progress |
| ImportError: jose | build | 1 | ✅ Resolved |
```

Resolved errors stay in the table for the session — they are part of the learning record.

---

## Writing to Session Log

At session end, every error (including resolved ones) goes into the session log's "What Failed" section with:
- The full error class and message
- Root cause identified
- Resolution applied
- Number of attempts

This is the primary input for the recursive learning corpus.
