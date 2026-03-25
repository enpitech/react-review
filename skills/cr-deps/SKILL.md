---
name: cr-deps
description: Run a dependency health audit checking for vulnerabilities, outdated packages, deprecations, license issues, and unused dependencies. Use when auditing project dependencies or checking supply chain security.
disable-model-invocation: true
---

# CR-Deps — Dependency Health Audit

Review scope: **All dependency manifests and lockfiles.**

Follow the audit criteria defined in `rules/deps.md`.

## Step 1 — Detect package manager

Check for the presence of:
- `package.json` + `package-lock.json` → npm
- `package.json` + `yarn.lock` → yarn
- `package.json` + `pnpm-lock.yaml` → pnpm
- `pyproject.toml` → poetry/uv/pip
- `requirements.txt` → pip
- `Pipfile` → pipenv

If multiple ecosystems are present (e.g., monorepo with Node + Python), audit each.

## Step 2 — Run vulnerability scan

- **npm**: `npm audit --json`
- **Python**: `pip-audit --format=json` (fall back to `pip-audit` if json flag unavailable)

Parse results and categorize by severity.

## Step 3 — Check for outdated packages

- **npm**: `npm outdated --json`
- **Python**: `pip list --outdated --format=json`

Identify packages that are major versions behind.

## Step 4 — Run remaining audit passes

Apply passes 3–6 from the criteria file:
- Deprecation check
- License compliance
- Unused dependencies (cross-reference manifests against source imports)
- Lockfile integrity

## Step 5 — Output findings

Collect all findings for `deps-audit-findings.md` in this format:

```
# Dependency Audit Findings

Generated: [date]
Package manager: [detected]
Total dependencies: [count]

## Pass 1 — VULNERABILITIES
(findings grouped by severity, or "✅ No known vulnerabilities")

## Pass 2 — OUTDATED PACKAGES
(findings or "✅ All packages up to date")

## Pass 3 — DEPRECATIONS
(findings or "✅ No deprecated packages")

## Pass 4 — LICENSE COMPLIANCE
(findings or "✅ No license issues")

## Pass 5 — UNUSED DEPENDENCIES
(findings or "✅ No unused dependencies detected")

## Pass 6 — LOCKFILE INTEGRITY
(findings or "✅ Lockfile is in sync")

## Summary
- Total findings: [count]
- CRITICAL: [count]
- WARNING: [count]
- Suggested fix commands listed below each finding
```

Each finding should follow this format:
```
### [SEVERITY] package@version

**Issue:** one-line description
**Details:** CVE ID, advisory URL, or specific concern
**Fix:** exact command to resolve (e.g., `npm install package@fixed-version`)
```

Show the user the findings.

## Step 6 — Autofix

Follow the autofix workflow defined in `rules/autofix.md`.

- **Local mode**: present the three options (create findings file as `deps-audit-findings.md`, fix step by step, fix all). For dependency fixes, apply only non-breaking fixes automatically; for breaking changes (major version bumps), list them and ask for confirmation.
- **CI mode**: post findings as PR comments with `/fix` and `/fix-all` reply instructions

- Run everything autonomously without asking to confirm each step (except the autofix choice in local mode).
