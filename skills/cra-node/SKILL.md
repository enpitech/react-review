---
name: cra-node
description: Run a full-codebase 7-pass Node.js audit with system-level checks and dependency audit. Use when auditing an entire Node.js project or performing a comprehensive backend code quality review.
disable-model-invocation: true
---

# CRA-Node — Full Codebase Node.js Audit

Review scope: **Entire codebase.** Local-only — not available in CI.

Follow the review criteria defined in `rules/node.md`.
Also apply the dependency audit criteria from `rules/deps.md` (see Step 4).

## Step 1 — Detect project context

Read `package.json` and check for:
- `express` dependency → enable Express-specific middleware checks
- `fastify` → enable Fastify plugin checks
- `koa` → enable Koa middleware checks
- `typescript` or `tsconfig.json` → skip checks covered by TS compiler
- `prisma`, `@prisma/client` → enable Prisma-specific query checks
- `sequelize` → enable Sequelize raw query checks
- `typeorm` → enable TypeORM query builder checks
- `mongoose` → enable Mongoose-specific checks

## Step 2 — Map the codebase

Build a mental model of the project:
- List all source files: `find . -name '*.ts' -o -name '*.js' -o -name '*.mjs' -o -name '*.cjs' | grep -v node_modules | head -200`
- Identify module boundaries, shared folders, and entry points
- Note the routing structure and middleware chain

## Step 3 — Run 7 review passes across all source files

Apply all 7 review passes and filtering rules from the criteria file across every source file.

Additionally, check for these **system-level issues**:

### System-Level Checks
- **Dead Exports**: exported functions/classes/types with zero consumers across the codebase
- **Circular Dependencies**: import/require cycles between modules
- **Duplicated Patterns**: same fetch/transform/response pattern repeated in 2+ route handlers — should be middleware or shared utility
- **Inconsistent Error Handling**: some modules use try/catch, others use .catch(), others ignore errors
- **Missing Graceful Shutdown**: no SIGTERM/SIGINT handlers for cleanup
- **Global State Pollution**: shared mutable state across modules that creates hidden coupling
- **Inconsistent Auth Patterns**: some routes use middleware, others check inline, some skip entirely

## Step 4 — Run dependency audit

Apply all 6 passes from `rules/deps.md` against the project's dependency manifests and lockfiles.
Append findings to the report under a "## Dependency Audit" section.

## Step 5 — Output findings

Write all findings to `cra-node-findings.md` in the project root, organized by pass:

```
# Node.js Full Audit Findings

Generated: [date]
Files scanned: [count]

## Pass 1 — BUGS
(findings or "✅ Clean")

## Pass 2 — SECURITY
(findings or "✅ Clean")

## Pass 3 — ASYNC PATTERNS
(findings or "✅ Clean")

## Pass 4 — ERROR HANDLING
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

Show the user the findings.

## Step 6 — Autofix

Follow the local mode autofix workflow defined in `rules/autofix.md`.

Present the three options (create findings file as `cra-node-findings.md`, fix step by step, fix all).
When fixing, apply CRITICAL fixes first, then WARNING.
- Do not flag issues that are clearly intentional based on git blame context.
- Run everything autonomously without asking to confirm each step (except the autofix choice).
