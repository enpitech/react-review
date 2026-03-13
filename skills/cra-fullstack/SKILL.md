---
name: cra-fullstack
description: Run a full-codebase cross-layer audit, auto-detecting languages and applying the appropriate criteria for each layer, plus system-level checks and dependency audit. Supports any combination of React, Node.js, Python, and other languages.
disable-model-invocation: true
---

# CRA-Fullstack — Full Codebase Fullstack Audit (Auto-Detect)

Review scope: **Entire codebase across all layers.** Local-only — not available in CI.

This skill dynamically detects which languages and frameworks are in the project, applies the matching criteria to each layer, runs cross-layer checks from `rules/fullstack.md`, and includes a full dependency audit from `rules/deps.md`.

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

Log detected stack at the top of findings.

## Step 2 — Map the full codebase

Build a mental model of the entire project:
- List all source files: `find . -type f | grep -v node_modules | grep -v .git | grep -v vendor | grep -v __pycache__ | grep -v .venv | head -300`
- Map frontend directories, backend directories, and shared directories
- Note the application architecture and module boundaries per layer

## Step 3 — Apply language-specific criteria per layer (full codebase)

For each detected layer, apply the review passes from the matching criteria file across **all source files** in that layer:

- **React files** → all 7 passes from `rules/react.md` + React system-level checks (dead exports, circular deps, architecture drift, bundle hotspots)
- **Node.js files** → all 7 passes from `rules/node.md` + Node system-level checks (dead exports, circular deps, inconsistent error handling, missing graceful shutdown)
- **Python files** → all 7 passes from `rules/python.md` + Python system-level checks (dead code, circular imports, missing migrations)
- **Other language files** → all 5 passes from `rules/general.md` + general system-level checks (dead code, circular deps, duplicated patterns)

## Step 4 — Run cross-layer checks

Apply all 7 cross-layer checks from `rules/fullstack.md`:
1. API Contract Validation
2. Shared Type Drift
3. Environment Variable Hygiene
4. Authentication Flow
5. Error Contract
6. Data Flow Security
7. API Versioning & Deprecation

## Step 5 — Run dependency audit

Apply all 6 passes from `rules/deps.md` against every dependency manifest in the project.
If monorepo, audit each workspace independently.
Append findings under a "## Dependency Audit" section.

## Step 6 — Output findings

Write all findings to `cra-fullstack-findings.md` in the project root:

```
# Fullstack Full Audit Findings

Generated: [date]
Detected stack: [e.g., "Vue (Nuxt) + Go (Gin)"]
Frontend files scanned: [count]
Backend files scanned: [count]
Shared files scanned: [count]

## Frontend ([detected framework])
### Review Passes
(findings per criteria pass, or "✅ Clean")
### System-Level Issues
(findings or "✅ Clean")

## Backend ([detected framework])
### Review Passes
(findings per criteria pass, or "✅ Clean")
### System-Level Issues
(findings or "✅ Clean")

## Cross-Layer Issues
### Check 1 — API Contract Validation
(findings or "✅ Clean")
...
### Check 7 — API Versioning & Deprecation
(findings or "✅ Clean")

## Dependency Audit
(findings or "✅ All dependencies healthy")

## Summary
- Total findings: [count]
- CRITICAL: [count] (frontend: X, backend: Y, cross-layer: Z, deps: W)
- WARNING: [count] (frontend: X, backend: Y, cross-layer: Z, deps: W)
- Top 5 files needing attention: [list]
```

Show the user the contents of the findings file.

## Step 7 — Offer to fix

Ask the user: "Would you like me to apply fixes? I can do all CRITICAL first, then WARNING."

If yes, apply CRITICAL fixes first (one at a time with user confirmation), then WARNING fixes.
- Do not flag issues that are clearly intentional based on git blame context.
- Run everything autonomously without asking to confirm each step (except the final fix offer).
