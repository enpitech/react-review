---
name: cra-general
description: Run a full-codebase 5-pass language-agnostic audit with system-level checks and dependency audit. Use when auditing an entire project in any language that doesn't have a dedicated skill (Vue.js, Go, Rust, Ruby, PHP, Java, etc.).
disable-model-invocation: true
---

# CRA-General — Full Codebase General Audit

Review scope: **Entire codebase.** Local-only — not available in CI.

Follow the review criteria defined in `rules/general.md`.
Also apply the dependency audit criteria from `rules/deps.md` (see Step 4).

## Step 1 — Detect project context

Auto-detect the language and framework by scanning:
- File extensions (`.vue`, `.go`, `.rs`, `.rb`, `.php`, `.java`, `.kt`, `.swift`, `.cs`, etc.)
- Config files: `package.json`, `go.mod`, `Cargo.toml`, `Gemfile`, `composer.json`, `build.gradle`, `pom.xml`, `pubspec.yaml`, etc.
- Read the relevant config file to identify frameworks and dependencies

State the detected language and framework at the top of your output.

## Step 2 — Map the codebase

Build a mental model of the project:
- List all source files: `find . -type f | grep -v node_modules | grep -v .git | grep -v vendor | grep -v __pycache__ | head -200`
- Identify module boundaries, package structure, and entry points
- Note the application architecture pattern (MVC, clean architecture, etc.)

## Step 3 — Run 5 review passes across all source files

Apply all 5 review passes and filtering rules from the criteria file across every source file.

Additionally, check for these **system-level issues**:

### System-Level Checks
- **Dead Code**: exported functions/classes/types with zero consumers across the codebase
- **Circular Dependencies**: import cycles between modules
- **Duplicated Patterns**: same logic repeated in 2+ distant files — should be shared
- **Architecture Drift**: modules violating the intended structure (e.g. direct DB access from handlers)
- **Inconsistent Patterns**: conflicting approaches for the same concern (3 different error handling strategies, etc.)
- **Configuration Issues**: dev settings in production config, missing environment-based config switching

## Step 4 — Run dependency audit

Apply all 6 passes from `rules/deps.md` against the project's dependency manifests and lockfiles.
Adapt to the detected package manager (npm, pip, go modules, cargo, bundler, composer, maven/gradle, etc.).
Append findings to the report under a "## Dependency Audit" section.

## Step 5 — Output findings

Write all findings to `cra-general-findings.md` in the project root, organized by pass:

```
# Full Audit Findings

Generated: [date]
Language: [detected]
Framework: [detected]
Files scanned: [count]

## Pass 1 — BUGS
(findings or "✅ Clean")

## Pass 2 — SECURITY
(findings or "✅ Clean")

## Pass 3 — ERROR HANDLING
(findings or "✅ Clean")

## Pass 4 — PERFORMANCE
(findings or "✅ Clean")

## Pass 5 — CODE QUALITY
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
