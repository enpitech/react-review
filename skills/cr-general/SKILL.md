---
name: cr-general
description: Run a 5-pass language-agnostic code review scoped to the current PR diff. Use when reviewing pull requests for any language or framework that doesn't have a dedicated skill (Vue.js, Go, Rust, Ruby, PHP, Java, etc.).
disable-model-invocation: true
---

# CR-General — Diff-Scoped General Code Review

Review scope: **PR diff + directly affected files only.**

Follow the review criteria defined in `rules/general.md`.

## Step 1 — Detect project context

Auto-detect the language and framework by scanning:
- File extensions in the diff (`.vue`, `.go`, `.rs`, `.rb`, `.php`, `.java`, `.kt`, `.swift`, `.cs`, etc.)
- Config files: `package.json`, `go.mod`, `Cargo.toml`, `Gemfile`, `composer.json`, `build.gradle`, `pom.xml`, `pubspec.yaml`, etc.
- Read the relevant config file to identify frameworks and dependencies

State the detected language and framework at the top of your output.

## Step 2 — Get PR context

Run these commands sequentially (no nested substitution):
1. `gh pr view --json number,title,body,baseRefName,headRefName,url` — store the `baseRefName` value
2. `git diff <baseRefName>...HEAD` — using the baseRefName from step 1
3. `git diff --name-only <baseRefName>...HEAD` — changed files list

## Step 3 — Expand affected scope

For each changed file, identify direct consumers (files that import from it) to catch breakage from interface changes. Only expand one level deep — do not crawl the entire dependency graph.

## Step 4 — Run review passes

Apply all 5 review passes and filtering rules from the criteria file against the diff and affected files. Adapt the checks to the detected language and framework.

## Step 5 — Output findings

Collect all findings with this format per issue:

```
### [SEVERITY] Pass N — file:line

**Issue:** one-line description
**Why:** reason you're confident this is a real issue
**Fix:** suggested code change
```

If no issues found: "✅ No issues found — all 5 review passes came back clean."

Show the user the findings.

## Step 6 — Autofix

Follow the autofix workflow defined in `rules/autofix.md`.

- **Local mode**: present the three options (create findings file as `cr-general-findings.md`, fix step by step, fix all)
- **CI mode**: post findings as PR comments with `/fix` and `/fix-all` reply instructions

- Do not flag issues that are clearly intentional based on git blame context.
- Run everything autonomously without asking to confirm each step (except the autofix choice in local mode).
