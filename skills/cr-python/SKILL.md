---
name: cr-python
description: Run a 7-pass Python code review scoped to the current PR diff. Use when reviewing pull requests for Python/Django/Flask/FastAPI projects, checking code before merge, or when the user asks for a Python code review.
disable-model-invocation: true
---

# CR-Python — Diff-Scoped Python Review

Review scope: **PR diff + directly affected files only.**

Follow the review criteria defined in `rules/python.md`.

## Step 1 — Detect project context

Check for project configuration and detect frameworks:
- `requirements.txt`, `pyproject.toml`, `Pipfile`, or `setup.py` → read dependencies
- `django` in dependencies → enable Django-specific checks (ORM, templates, middleware, settings)
- `flask` in dependencies → enable Flask security checks
- `fastapi` in dependencies → enable FastAPI checks (dependency injection, async patterns)
- `sqlalchemy` in dependencies → enable SQLAlchemy query checks
- `pydantic` in dependencies → leverage model validation, reduce redundant input validation checks
- `mypy` or `pyright` in config → reduce type safety pass to runtime-only issues
- `pytest` in devDependencies → skip test-file patterns (assert statements, fixtures)

## Step 2 — Get PR context

Run these commands sequentially (no nested substitution):
1. `gh pr view --json number,title,body,baseRefName,headRefName,url` — store the `baseRefName` value
2. `git diff <baseRefName>...HEAD` — using the baseRefName from step 1
3. `git diff --name-only <baseRefName>...HEAD` — changed files list

## Step 3 — Expand affected scope

For each changed `.py` file, identify direct consumers (files that import from it) to catch breakage from interface changes. Only expand one level deep — do not crawl the entire dependency graph.

## Step 4 — Run review passes

Apply all 7 review passes and filtering rules from the criteria file against the diff and affected files.

## Step 5 — Output findings

Write all findings to a temporary file `cr-python-findings.md` in the project root with this format per issue:

```
### [SEVERITY] Pass N — file:line

**Issue:** one-line description
**Why:** reason you're confident this is a real issue
**Fix:** suggested code change
```

If no issues found, write: "✅ No issues found — all 7 review passes came back clean."

Show the user the contents of the findings file.

## Step 6 — Offer to fix

Ask the user: "Would you like me to apply the suggested fixes?"

If yes, apply each fix one at a time, showing the change before moving to the next.
Do NOT apply fixes without user confirmation.
- Do not flag issues that are clearly intentional based on git blame context.
- Run everything autonomously without asking to confirm each step.
