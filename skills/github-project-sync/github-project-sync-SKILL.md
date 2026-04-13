---
name: github-project-sync
description: Bootstraps or adopts a project into the agent system. BOOTSTRAP mode: initializes a brand new project with GitHub sync, branch protection, pre-commit hooks, and manifest files. ADOPT mode: brings an existing git repo into the agent system without touching git history, remotes, or branches. Invoke whenever agent.md or architecture.md are missing regardless of whether a git repo already exists.
---

# GitHub Project Sync

Initializes the agent system into a project — either from scratch or by adopting an existing repo.

Read the orchestrator's detected state before starting. The mode you run depends on what was found:
- **BOOTSTRAP mode** — no `.git/` folder exists. Build everything from scratch.
- **ADOPT mode** — `.git/` folder exists but manifests are missing. Generate manifests from observed reality, do not touch git infrastructure.

---

## BOOTSTRAP MODE — Brand New Project

### What This Creates

```
project-root/
├── .git/
│   └── hooks/
│       └── pre-commit          ← blocks commits missing session logs
├── .gitignore                  ← sensible defaults for detected language
├── sessions/
│   └── session_YYYYMMDD_HHMMSS_init.md   ← first session log
├── architecture.md             ← filled from template
├── agent.md                    ← filled from template
└── README.md                   ← minimal stub if not present
```

### Bootstrap Step 1 — Gather Project Info

Before running anything, collect from the human:

- `PROJECT_NAME` — slug used for repo name (kebab-case)
- `PROJECT_PURPOSE` — one sentence describing what this project does
- `LANGUAGES` — primary language(s)
- `FRAMEWORK` — runtime or framework (e.g. FastAPI, Next.js, none)
- `GITHUB_USER_OR_ORG` — GitHub account for the new repo
- `BRANCH_DEFAULT` — almost always `main`

If any are unknown, ask the human before proceeding.

### Bootstrap Step 2 — Initialize Git & GitHub Repo

```bash
# In the project directory:
git init
git checkout -b main

# Create GitHub repo (requires gh CLI authenticated)
gh repo create {{GITHUB_USER_OR_ORG}}/{{PROJECT_NAME}} \
  --private \
  --source=. \
  --remote=origin \
  --push

# Create develop branch
git checkout -b develop
git push -u origin develop
git checkout main
```

If `gh` is not available or not authenticated, pause and instruct the human to run:
```
gh auth login
```
Then resume from where you left off.

### Bootstrap Step 3 — Install Pre-Commit Hook

Create `.git/hooks/pre-commit` with the content below and make it executable:

```bash
#!/bin/bash
# pre-commit: enforces session log exists before any commit
# Installed by github-project-sync skill

SESSION_DIR="sessions"
TODAY=$(date +%Y%m%d)

# Check for a session log from today's session
if ! ls "$SESSION_DIR"/session_${TODAY}_*.md 1>/dev/null 2>&1; then
  echo ""
  echo "❌ COMMIT BLOCKED: No session log found for today."
  echo "   Write a session log to sessions/session_$(date +%Y%m%d_%H%M%S).md"
  echo "   then retry your commit."
  echo ""
  exit 1
fi

echo "✅ Session log found. Proceeding with commit."
exit 0
```

```bash
chmod +x .git/hooks/pre-commit
```

### Bootstrap Step 4 — Generate .gitignore

Detect the project language and write an appropriate `.gitignore`. Minimum contents:

```
# Session logs are committed (they are part of the learning corpus)
# Do NOT add sessions/ here

# Environment
.env
.env.*
!.env.example

# OS
.DS_Store
Thumbs.db

# Editor
.vscode/
.idea/
*.swp

# Language-specific (append detected language defaults below)
```

Append language-specific patterns based on detected stack (Python, Node, Rust, Go, etc.).

### Bootstrap Step 5 — Fill Architecture Template

Read `templates/architecture.template.md` and substitute all `{{PLACEHOLDERS}}`:

- `{{PROJECT_NAME}}` → project name
- `{{TIMESTAMP}}` → current ISO timestamp
- `{{SESSION_ID}}` → `session_YYYYMMDD_HHMMSS` format
- `{{PROJECT_PURPOSE}}` → collected in Step 1
- `{{LANGUAGES}}` → collected in Step 1
- `{{FRAMEWORK}}` → collected in Step 1
- `{{GITHUB_REPO_URL}}` → from Step 2 output
- `{{REPO_ROOT}}` → project directory name
- `{{STRUCTURE_PLACEHOLDER}}` → run `tree -L 2` and paste output
- All table rows → stub with `TBD` where unknown

Write filled file to `architecture.md` in project root.

### Bootstrap Step 6 — Fill Agent State Template

Read `templates/agent.template.md` and substitute placeholders:

- `{{PHASE}}` → `INIT`
- `{{ACTIVE_AGENT}}` → `orchestrator`
- `{{BRANCH}}` → `develop`
- `{{LAST_GREEN_SHA}}` → output of `git rev-parse HEAD`
- `{{TASK_GOAL}}` → `Initial project setup`
- `{{NEXT_ACTION}}` → `READ architecture.md then begin first task decomposition`
- All other fields → appropriate stubs

Write filled file to `agent.md` in project root.

### Bootstrap Step 7 — Create Sessions Directory and First Log

```bash
mkdir -p sessions
touch sessions/.gitkeep
```

Write the first session log to `sessions/session_YYYYMMDD_HHMMSS_init.md` using the `session-memory` skill format. Document what was created and what the human told you about the project.

### Bootstrap Step 8 — Initial Commit

```bash
git add architecture.md agent.md sessions/ .gitignore README.md
git commit -m "chore(init): bootstrap project with agent system manifests

- architecture.md: codebase map template
- agent.md: session state machine initialized (Phase: INIT)
- sessions/: learning corpus directory with first session log
- pre-commit hook: enforces session log before commits
- .gitignore: language-appropriate defaults"

git push origin develop
```

### Bootstrap Completion Checklist

- [ ] GitHub repo created and accessible
- [ ] `main` and `develop` branches exist and are pushed
- [ ] Pre-commit hook installed and executable
- [ ] `architecture.md` filled and committed
- [ ] `agent.md` filled with Phase: INIT and committed
- [ ] `sessions/` directory exists with first session log committed
- [ ] `.gitignore` appropriate for project language
- [ ] Orchestrator confirms it can read both manifest files

---

## ADOPT MODE — Existing Project

Use this mode when `.git/` already exists but `agent.md` and `architecture.md` do not. The project was started before the agent system was installed.

**CRITICAL: Do NOT run in adopt mode:**
- `git init`
- `gh repo create`
- Any branch creation or deletion
- Any changes to existing remotes
- Any force-push or history rewrite

### Adopt Step 1 — Scan the Existing Repo

Run these read-only commands and hold the output in context. This is your source of truth for generating the manifests:

```bash
git remote -v                          # existing remote URL(s)
git branch -a                          # all local and remote branches
git log --oneline -10                  # recent commit history
git rev-parse HEAD                     # current HEAD SHA
git branch --show-current              # active branch name
tree -L 2                              # directory structure
cat package.json 2>/dev/null           # Node project info
cat requirements.txt 2>/dev/null       # Python dependencies
cat pyproject.toml 2>/dev/null         # Python project info
cat Cargo.toml 2>/dev/null             # Rust project info
cat go.mod 2>/dev/null                 # Go project info
ls sessions/ 2>/dev/null               # prior session logs if any exist
```

### Adopt Step 2 — Fill Architecture Template from Scan

Read `templates/architecture.template.md` and substitute placeholders using the scan output — not stubs. Use real values wherever possible:

- `{{PROJECT_NAME}}` → directory name or name from package.json/pyproject.toml
- `{{TIMESTAMP}}` → current ISO timestamp
- `{{SESSION_ID}}` → `session_YYYYMMDD_HHMMSS_adoption`
- `{{PROJECT_PURPOSE}}` → infer from README if present, otherwise ask human for one sentence
- `{{LANGUAGES}}` → detected from project files
- `{{FRAMEWORK}}` → detected from dependencies
- `{{GITHUB_REPO_URL}}` → from `git remote -v` output
- `{{REPO_ROOT}}` → current directory name
- `{{STRUCTURE_PLACEHOLDER}}` → output of `tree -L 2`
- Dependency table → populated from detected dependency files
- Change Log → first entry: "Agent system adopted into existing project"

Write filled file to `architecture.md` in project root.

### Adopt Step 3 — Fill Agent State Template from Scan

Read `templates/agent.template.md` and substitute with real observed values:

- `{{PHASE}}` → `ADOPTED`
- `{{ACTIVE_AGENT}}` → `orchestrator`
- `{{BRANCH}}` → output of `git branch --show-current`
- `{{LAST_GREEN_SHA}}` → output of `git rev-parse HEAD`
- `{{TASK_GOAL}}` → `Adopt existing project into agent system`
- `{{NEXT_ACTION}}` → `Review architecture.md for accuracy, then ask human for first task`
- All hook statuses → `N/A` for this adoption session except session log

Write filled file to `agent.md` in project root.

### Adopt Step 4 — Install Pre-Commit Hook (conditional)

Check whether a pre-commit hook already exists:

```bash
ls .git/hooks/pre-commit 2>/dev/null
```

- **If it does NOT exist:** Install the hook exactly as in Bootstrap Step 3.
- **If it already exists:** Read its contents and report to the human what it does. Do not overwrite it. Add a note in `agent.md` under Known Constraints that the pre-commit hook was pre-existing and session log enforcement may not be active.

### Adopt Step 5 — Create Sessions Directory (conditional)

```bash
# Only create if it does not already exist
mkdir -p sessions
# Only create .gitkeep if sessions/ is empty
[ -z "$(ls -A sessions/)" ] && touch sessions/.gitkeep
```

### Adopt Step 6 — Write Adoption Session Log

Write a session log to `sessions/session_YYYYMMDD_HHMMSS_adoption.md` using the `session-memory` skill format.

The "What Was Done" section must document:
- What the repo scan found (branches, recent commits, detected language/framework)
- What was generated (architecture.md, agent.md)
- What the agent system does not yet know about this project (gaps to fill in future sessions)
- Any pre-existing hooks or tooling that may interact with agent workflows

### Adopt Step 7 — Commit Manifest Files Only

Stage and commit only the new files. Do not touch any existing files:

```bash
git add agent.md architecture.md sessions/
git commit -m "chore(agent-system): adopt existing project into agent workflow

- architecture.md: generated from repo scan ($(date +%Y-%m-%d))
- agent.md: session state machine initialized (Phase: ADOPTED)
- sessions/: learning corpus directory created with adoption log
- pre-commit hook: installed / pre-existing (see agent.md constraints)"
```

Push to the current branch:
```bash
git push origin $(git branch --show-current)
```

### Adopt Completion Checklist

- [ ] Repo scanned — remote, branches, recent history captured
- [ ] `architecture.md` generated from real observed values (not stubs)
- [ ] `agent.md` generated with Phase: ADOPTED, real branch and SHA
- [ ] Pre-commit hook installed or pre-existing noted in `agent.md`
- [ ] `sessions/` directory exists with adoption session log
- [ ] Manifests committed and pushed on existing branch
- [ ] Orchestrator confirms it can read both manifest files
- [ ] Human informed of any gaps or unknowns noted in adoption log
