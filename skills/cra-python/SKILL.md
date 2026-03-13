---
name: cra-python
description: Run a full-codebase 7-pass Python audit with system-level checks and dependency audit. Use when auditing an entire Python project or performing a comprehensive code quality review.
disable-model-invocation: true
---

# CRA-Python — Full Codebase Python Audit

Review scope: **Entire codebase.** Local-only — not available in CI.

Follow the review criteria defined in `rules/python.md`.
Also apply the dependency audit criteria from `rules/deps.md` (see Step 4).

## Step 1 — Detect project context

Check for project configuration and detect frameworks:
- `requirements.txt`, `pyproject.toml`, `Pipfile`, or `setup.py` → read dependencies
- `django` in dependencies → enable Django-specific checks
- `flask` in dependencies → enable Flask security checks
- `fastapi` in dependencies → enable FastAPI checks
- `sqlalchemy` in dependencies → enable SQLAlchemy query checks
- `pydantic` in dependencies → leverage model validation
- `mypy` or `pyright` in config → reduce type safety pass to runtime-only issues
- `pytest` in devDependencies → skip test-file patterns

## Step 2 — Map the codebase

Build a mental model of the project:
- List all source files: `find . -name '*.py' | grep -v __pycache__ | grep -v .venv | grep -v venv | head -200`
- Identify package structure, module boundaries, and entry points
- Note the app structure (Django apps, Flask blueprints, FastAPI routers)

## Step 3 — Run 7 review passes across all source files

Apply all 7 review passes and filtering rules from the criteria file across every source file.

Additionally, check for these **system-level issues**:

### System-Level Checks
- **Dead Code**: exported functions/classes never imported elsewhere in the codebase
- **Circular Imports**: import cycles between modules
- **Duplicated Patterns**: same query/transform/response pattern repeated in 2+ views — should be shared
- **Inconsistent Patterns**: mixing sync and async without reason, different serialization approaches
- **Missing `__init__.py`**: packages missing init files (for non-namespace packages)
- **Configuration Drift**: dev settings leaking into production config, hardcoded environment values
- **Missing Migrations**: model changes without corresponding migration files (Django)

## Step 4 — Run dependency audit

Apply all 6 passes from `rules/deps.md` against the project's dependency manifests and lockfiles.
Append findings to the report under a "## Dependency Audit" section.

## Step 5 — Output findings

Write all findings to `cra-python-findings.md` in the project root, organized by pass:

```
# Python Full Audit Findings

Generated: [date]
Files scanned: [count]

## Pass 1 — BUGS
(findings or "✅ Clean")

## Pass 2 — SECURITY
(findings or "✅ Clean")

## Pass 3 — TYPE SAFETY
(findings or "✅ Clean")

## Pass 4 — ASYNC PATTERNS
(findings or "✅ Clean")

## Pass 5 — API DESIGN
(findings or "✅ Clean")

## Pass 6 — PERFORMANCE
(findings or "✅ Clean")

## Pass 7 — CODE QUALITY
(findings or "✅ Clean")

## System-Level Issues
(findings or "✅ Clean")

## Dependency Audit
(findings or "✅ All dependencies healthy")

## Summary
- Total findings: [count]
- CRITICAL: [count]
- WARNING: [count]
- Top 3 files needing attention: [list]
```

Show the user the contents of the findings file.

## Step 6 — Offer to fix

Ask the user: "Would you like me to apply fixes? I can do all CRITICAL first, then WARNING."

If yes, apply CRITICAL fixes first (one at a time with user confirmation), then WARNING fixes.
- Do not flag issues that are clearly intentional based on git blame context.
- Run everything autonomously without asking to confirm each step (except the final fix offer).
