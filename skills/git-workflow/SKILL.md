---
name: git-workflow
description: Enforces safe git operations including branching, conventional commits, pull-before-push, merge conflict detection, and release tagging. Invoke before any commit, merge, or push operation. Orchestrator-scoped — only the orchestrator calls git commands directly.
---

# Git Workflow

The orchestrator is the only agent that runs git commands. All other agents produce work; the orchestrator commits it.

---

## Branch Model

```
main          ← protected, production-ready, human-approved merges only
  └── develop ← integration, always passing tests
        ├── feature/short-description   ← new functionality
        ├── fix/short-description       ← bug fixes  
        └── hotfix/short-description    ← urgent fixes direct from main
```

### Branch Naming Rules
- Use kebab-case: `feature/add-auth-middleware`
- Keep it short and descriptive: max 5 words after the prefix
- Always branch from `develop` (except hotfix → branches from `main`)

### Creating a Branch
```bash
git checkout develop
git pull origin develop          # always pull first
git checkout -b feature/{{name}}
git push -u origin feature/{{name}}
```

---

## Commit Protocol

### Pre-Commit Checklist (run in order)
- [ ] Tests pass: `{{test command}}`
- [ ] Session log exists in `sessions/` for today
- [ ] `architecture.md` updated if structure changed
- [ ] `agent.md` hook table complete
- [ ] No debug prints or temporary code left in
- [ ] No secrets or API keys in staged files: `git diff --cached | grep -i 'key\|secret\|password\|token'`

### Commit Message Format (Conventional Commits)

```
type(scope): short imperative description

[optional body explaining WHY, not what]

[optional footer: Breaking changes, issue refs]
```

**Types:**
- `feat` — new feature
- `fix` — bug fix
- `refactor` — code change that neither fixes a bug nor adds a feature
- `test` — adding or updating tests
- `docs` — documentation only
- `chore` — build process, dependency updates, tooling
- `perf` — performance improvement
- `style` — formatting only (no logic change)

**Examples:**
```
feat(auth): add JWT refresh token endpoint

fix(parser): handle empty string input without raising ValueError

refactor(api): extract request validation into middleware

test(auth): add integration tests for token expiry edge cases

chore(deps): bump fastapi from 0.103.0 to 0.104.1
```

### Running the Commit
```bash
git add {{files}}                # stage explicitly, never `git add .` blindly
git diff --cached                # review what is staged
git commit -m "{{message}}"
git push origin {{branch}}
```

---

## Merging Protocol

### Feature → Develop
Only merge when:
- All acceptance criteria in `agent.md` are met
- Tests pass on the feature branch
- Session log written and committed on the branch

```bash
git checkout develop
git pull origin develop
git merge --no-ff feature/{{name}} -m "merge(feature/{{name}}): {{summary}}"
git push origin develop
git branch -d feature/{{name}}           # clean up local
git push origin --delete feature/{{name}} # clean up remote
```

### Develop → Main
**Requires explicit human approval. Do not run without it.**

```bash
git checkout main
git pull origin main
git merge --no-ff develop -m "release(v{{version}}): {{summary}}"
git tag -a v{{version}} -m "Release v{{version}}: {{summary}}"
git push origin main
git push origin --tags
```

---

## Pull Before Push (Always)

Before any push, pull to detect conflicts early:
```bash
git pull --rebase origin {{branch}}
```

If rebase fails due to conflicts:
1. List conflicted files: `git diff --name-only --diff-filter=U`
2. For each conflict: assess whether `build` or `refactor` agent should resolve
3. Route resolution task to appropriate agent
4. After resolution: `git add {{file}}` then `git rebase --continue`
5. If too complex: `git rebase --abort` → report to human

---

## Release Tagging

Tag every merge to `main`:
```bash
git tag -a v{{MAJOR}}.{{MINOR}}.{{PATCH}} -m "{{release notes summary}}"
git push origin --tags
```

Use semantic versioning:
- `MAJOR` — breaking changes
- `MINOR` — new features, backward compatible
- `PATCH` — bug fixes

---

## Emergency Procedures

### Undo Last Commit (not yet pushed)
```bash
git reset --soft HEAD~1          # keeps changes staged
```

### Undo Last Commit (already pushed) — requires human approval
```bash
git revert HEAD                  # creates a new undo commit, safe for shared branches
git push origin {{branch}}
```

### Complete Rollback → See `rollback-recovery` skill
