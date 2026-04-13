---
name: dependency-audit
description: Audits new dependencies for version conflicts, known vulnerabilities, and license compatibility before any install or import is added to the project. Invoke before any `pip install`, `npm install`, `cargo add`, or equivalent. Scoped to build and refactor agents.
---

# Dependency Audit

No package gets added to this project without passing audit. One bad dependency can brick the environment, introduce a vulnerability, or create a license conflict that blocks deployment.

---

## Trigger Conditions

Invoke before:
- Any `pip install {{package}}`
- Any `npm install {{package}}`
- Any `cargo add {{package}}`
- Any manual edit to `requirements.txt`, `package.json`, `Cargo.toml`, or equivalent
- Upgrading an existing dependency to a new major version

---

## Audit Protocol

### Step 1 — Check for Existing Coverage

Before adding anything new, check if the functionality already exists:

```bash
# Python
pip list | grep -i {{keyword}}
cat requirements.txt

# Node
cat package.json | grep -i {{keyword}}
ls node_modules/ | grep -i {{keyword}}
```

If the functionality is already available via an installed package, use it. Do not add a duplicate.

### Step 2 — Version Conflict Check

```bash
# Python — check for conflicts before installing
pip install {{package}}=={{version}} --dry-run 2>&1

# Node
npm install {{package}} --dry-run 2>&1

# Check currently pinned versions for conflicts
pip check       # Python: checks installed packages for incompatibilities
```

If conflicts are detected: do NOT proceed. Report to orchestrator with the conflict details. Do not attempt to resolve version conflicts autonomously — they can cascade.

### Step 3 — Vulnerability Check

```bash
# Python
pip install safety --quiet
safety check --package {{package}}=={{version}}

# Node
npm audit

# If safety/audit tools not available:
# Search: "{{package}} {{version}} CVE" in session notes
# Check: https://pypi.org/project/{{package}}/ (release history, yanked versions)
```

**Block install if:** any HIGH or CRITICAL severity CVE is open and unpatched.
**Proceed with warning if:** LOW or MEDIUM CVE exists — log it in `architecture.md` dependency table with ⚠️ status.

### Step 4 — License Compatibility Check

Check the package license against project requirements:

| License | Generally Compatible | Notes |
|---|---|---|
| MIT | ✅ Yes | Permissive, no restrictions |
| Apache 2.0 | ✅ Yes | Permissive, patent grant included |
| BSD 2/3-Clause | ✅ Yes | Permissive |
| ISC | ✅ Yes | Permissive |
| LGPL | ⚠️ Usually | Check if linking is static or dynamic |
| GPL | ❌ Caution | Copyleft — may affect distribution |
| AGPL | ❌ Caution | Copyleft — applies to network use |
| Proprietary | ❌ Stop | Requires human decision |

```bash
# Python
pip show {{package}} | grep License

# Node
npm info {{package}} license
```

If license is restrictive or unclear: pause and ask the human before proceeding.

### Step 5 — Pin the Version

Never install without pinning:

```bash
# Find the exact installed version
pip show {{package}} | grep Version

# Add to requirements.txt with pin
echo "{{package}}=={{version}}" >> requirements.txt

# Node — use exact version in package.json (no ^ or ~)
npm install {{package}}@{{version}} --save-exact
```

### Step 6 — Update Architecture.md

Add or update the dependency in `architecture.md` Dependencies table:

```markdown
| {{package}} | {{version}} | {{purpose — one sentence}} | ✅ Audited {{date}} |
```

---

## Audit Result Format

Report audit results to the orchestrator before proceeding:

```
DEPENDENCY AUDIT — {{package}} {{version}}
==========================================
Existing coverage:    None found / {{alternative}}
Version conflicts:    None / {{details}}
Vulnerabilities:      None / {{CVE details}}
License:              {{license}} — ✅ Compatible / ⚠️ Review / ❌ Block
Recommendation:       ✅ APPROVED / ⚠️ APPROVED WITH WARNING / ❌ BLOCKED

Reason: {{brief explanation if not a clean approval}}
```

Only proceed with install after orchestrator acknowledges the audit result.
