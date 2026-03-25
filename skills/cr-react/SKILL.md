---
name: cr-react
description: Run a 7-pass React/Next.js code review scoped to the current PR diff. Use when reviewing pull requests, checking code before merge, or when the user asks for a React code review.
disable-model-invocation: true
---

# CR-React — Diff-Scoped React Review

Review scope: **PR diff + directly affected files only.**

Follow the review criteria defined in `rules/react.md`.

## Step 1 — Detect project context

Read `package.json` and check for:
- `next` dependency → enable SSR/RSC rules (Pass 2 Server Actions, Pass 5 Suspense/serialization)
- `babel-plugin-react-compiler` or `react-compiler` → skip unnecessary memoization findings
- `@radix-ui`, `shadcn`, or similar → flag excessive inline styling more aggressively

## Step 2 — Get PR context

Run these commands sequentially (no nested substitution):
1. `gh pr view --json number,title,body,baseRefName,headRefName,url` — store the `baseRefName` value
2. `git diff <baseRefName>...HEAD` — using the baseRefName from step 1
3. `git diff --name-only <baseRefName>...HEAD` — changed files list

## Step 3 — Expand affected scope

For each changed file, identify direct consumers (files that import from it) to catch breakage from interface changes. Only expand one level deep — do not crawl the entire dependency graph.

## Step 4 — Run review passes

Apply all 7 review passes and filtering rules from the criteria file against the diff and affected files.

## Step 5 — Output findings

Collect all findings with this format per issue:

```
### [SEVERITY] Pass N — file:line

**Issue:** one-line description
**Why:** reason you're confident this is a real issue
**Fix:** suggested code change
```

If no issues found: "✅ No issues found — all 7 review passes came back clean."

Show the user the findings.

## Step 6 — Autofix

Follow the autofix workflow defined in `rules/autofix.md`.

- **Local mode**: present the three options (create findings file as `cr-react-findings.md`, fix step by step, fix all)
- **CI mode**: post findings as PR comments with `/fix` and `/fix-all` reply instructions

- Do not flag issues that are clearly intentional based on git blame context.
- Run everything autonomously without asking to confirm each step (except the autofix choice in local mode).
