---
name: cr-fullstack
description: Run a cross-layer code review on a fullstack PR diff, auto-detecting languages and applying the appropriate criteria for each layer. Supports any combination of React, Node.js, Python, and other languages.
disable-model-invocation: true
---

# CR-Fullstack — Diff-Scoped Fullstack Review (Auto-Detect)

Review scope: **PR diff + directly affected files across all layers.**

This skill dynamically detects which languages and frameworks are in the project, applies the matching criteria to each layer, and runs cross-layer checks from `rules/fullstack.md`.

## Step 1 — Detect project stack

Scan the project root to identify all layers. Check:

**Frontend detection:**
- `package.json` with `react`/`next` → **React** layer → use `rules/react.md`
- `package.json` with `vue`/`nuxt` → **Vue** layer → use `rules/general.md`
- `package.json` with `@angular/core` → **Angular** layer → use `rules/general.md`
- `package.json` with `svelte`/`@sveltejs/kit` → **Svelte** layer → use `rules/general.md`

**Backend detection:**
- `package.json` with `express`/`fastify`/`koa`/`hapi` (and no React) → **Node.js** layer → use `rules/node.md`
- `pyproject.toml`/`requirements.txt` with `django`/`flask`/`fastapi` → **Python** layer → use `rules/python.md`
- `go.mod` exists → **Go** layer → use `rules/general.md`
- `Gemfile` with `rails`/`sinatra` → **Ruby** layer → use `rules/general.md`
- `composer.json` with `laravel` → **PHP** layer → use `rules/general.md`
- `Cargo.toml` exists → **Rust** layer → use `rules/general.md`
- `build.gradle`/`pom.xml` → **Java/Kotlin** layer → use `rules/general.md`

**Monorepo detection:**
- Check for `workspaces` in root `package.json`, `lerna.json`, `nx.json`, `turbo.json`
- If monorepo, read each workspace's `package.json` to classify

Log detected stack at the top of findings (e.g., "Detected: React frontend + Python backend").

## Step 2 — Get PR context

Run these commands sequentially (no nested substitution):
1. `gh pr view --json number,title,body,baseRefName,headRefName,url` — store the `baseRefName` value
2. `git diff <baseRefName>...HEAD` — using the baseRefName from step 1
3. `git diff --name-only <baseRefName>...HEAD` — changed files list

Classify each changed file by layer:
- **Frontend**: `.tsx`, `.jsx`, `.vue`, `.svelte`, `.component.ts` files, or files in frontend directories
- **Backend**: `.py`, `.go`, `.rb`, `.php`, `.java`, `.rs` files, or `.ts`/`.js` files in backend directories
- **Shared**: files in `shared/`, `common/`, `types/`, `packages/shared/`
- **Config**: config files, CI files, docker files
- **Dependency**: `package.json`, `requirements.txt`, lockfiles

## Step 3 — Apply language-specific criteria per layer

For each detected layer, apply the review passes from the matching criteria file:

- **React files** → all 7 passes from `rules/react.md`
- **Node.js files** → all 7 passes from `rules/node.md`
- **Python files** → all 7 passes from `rules/python.md`
- **Other language files** → all 5 passes from `rules/general.md`

Include context-aware adjustments from each criteria file.

## Step 4 — Run cross-layer checks

Apply all 7 cross-layer checks from `rules/fullstack.md`:
1. API Contract Validation
2. Shared Type Drift
3. Environment Variable Hygiene
4. Authentication Flow
5. Error Contract
6. Data Flow Security
7. API Versioning & Deprecation

## Step 5 — Output findings

Write all findings to `cr-fullstack-findings.md` in the project root:

```
# Fullstack Review Findings

Generated: [date]
Detected stack: [e.g., "React + Python (FastAPI)"]
Frontend files reviewed: [count]
Backend files reviewed: [count]
Shared files reviewed: [count]

## Frontend ([detected framework])
(findings per criteria pass, or "✅ Clean")

## Backend ([detected framework])
(findings per criteria pass, or "✅ Clean")

## Cross-Layer Issues
### Check 1 — API Contract Validation
(findings or "✅ Clean")
...
### Check 7 — API Versioning & Deprecation
(findings or "✅ Clean")

## Summary
- Total findings: [count]
- CRITICAL: [count] (frontend: X, backend: Y, cross-layer: Z)
- WARNING: [count] (frontend: X, backend: Y, cross-layer: Z)
- Top 3 files needing attention: [list]
```

Show the user the contents of the findings file.

## Step 6 — Offer to fix

Ask the user: "Would you like me to apply fixes? I can do all CRITICAL first, then WARNING."

If yes, apply CRITICAL fixes first (one at a time with user confirmation), then WARNING fixes.
- Do not flag issues that are clearly intentional based on git blame context.
- Run everything autonomously without asking to confirm each step (except the final fix offer).
