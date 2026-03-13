# Code Review

Multi-language code review [plugin](https://code.claude.com/docs/en/plugins) for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) — CI + local [skills](https://code.claude.com/docs/en/skills).

Supports **React/Next.js**, **Node.js**, **Python**, **any language** (general), **dependency auditing**, and **fullstack** cross-layer reviews with auto-detection.

## Architecture

Two scope prefixes, applied uniformly across all languages:

| Prefix | Scope | Where | Includes deps? |
|--------|-------|-------|-----------------|
| `cr-` | PR diff + affected files | CI + Local | No |
| `cra-` | Full codebase audit | Local only | Yes |

## Skills Overview

| Skill | Scope | What it does | CI Trigger |
|-------|-------|--------------|------------|
| `cr-general` | Diff | 5-pass language-agnostic review | `/cr-general` |
| `cra-general` | Full | General audit + system checks + dep audit | Local only |
| `cr-fullstack` | Diff | Auto-detect stack + cross-layer checks | `/cr-fullstack` |
| `cra-fullstack` | Full | Full audit per layer + cross-layer + dep audit | Local only |
| `cr-react` | Diff | 7-pass React/Next.js review | `/cr-react` or `/claude-review` |
| `cra-react` | Full | React audit + system checks + dep audit | Local only |
| `cr-node` | Diff | 7-pass Node.js review | `/cr-node` |
| `cra-node` | Full | Node.js audit + system checks + dep audit | Local only |
| `cr-python` | Diff | 7-pass Python review | `/cr-python` |
| `cra-python` | Full | Python audit + system checks + dep audit | Local only |
| `cr-deps` | Deps | 6-pass dependency health audit | `/cr-deps` |

All skills report only **CRITICAL** (will break in prod) and **WARNING** (likely bug/security risk) at **8/10+ confidence**. No noise.

---

## CR-React — React/Next.js

7 sequential review passes:

| Pass | Focus | Examples |
|------|-------|---------|
| 1. BUGS | Logic errors | null access, race conditions, wrong conditionals |
| 2. SECURITY | Vulnerabilities | XSS, exposed secrets, unauthenticated Server Actions |
| 3. COMPONENT ARCHITECTURE | Structure | God components, prop drilling, cross-feature imports |
| 4. HOOKS & STATE | React patterns | Derived state in useEffect, stale closures, joinable hooks |
| 5. PERFORMANCE | Speed | Sequential awaits, missing dynamic imports, barrel file imports |
| 6. CODE QUALITY | React-specific | Array mutation, missing error boundaries, duplicated logic |
| 7. INTENT CHECK | PR scope | Unrelated changes that snuck into the diff |

**Context-aware**: React Compiler, Next.js SSR/RSC, design systems (`@radix-ui`, `shadcn`).

| Command | Scope |
|---------|-------|
| `/enpitech:cr-react` | PR diff review |
| `/enpitech:cra-react` | Full codebase audit + system checks + dep audit |

---

## CR-Node — Node.js

7 sequential review passes based on **OWASP Node.js Security Cheat Sheet** and **eslint-plugin-security**:

| Pass | Focus | Examples |
|------|-------|---------|
| 1. BUGS | Logic errors | Unhandled promise rejections, race conditions, event loop blocking |
| 2. SECURITY | Vulnerabilities | eval/exec injection, prototype pollution, ReDoS, SSRF, missing helmet/CSRF |
| 3. ASYNC PATTERNS | Event loop | Blocking calls in async, missing Promise.all, stream backpressure |
| 4. ERROR HANDLING | Resilience | Bare catch, missing error events, uncaughtException without exit |
| 5. API DESIGN | Express/Fastify | Missing request size limits, rate limiting, input validation, permissive CORS |
| 6. PERFORMANCE | Runtime | Sync fs/crypto ops, missing connection pooling, N+1 queries, memory leaks |
| 7. CODE QUALITY | Node-specific | `require(variable)`, `new Buffer()`, deprecated APIs, missing graceful shutdown |

**Context-aware**: Express, Fastify, Koa; TypeScript; Prisma, Sequelize, TypeORM, Mongoose.

| Command | Scope |
|---------|-------|
| `/enpitech:cr-node` | PR diff review |
| `/enpitech:cra-node` | Full codebase audit + system checks + dep audit |

---

## CR-Python — Python

7 sequential review passes based on **Bandit** (B1xx–B7xx), **Ruff/flake8-bandit**, and **Pylint**:

| Pass | Focus | Examples |
|------|-------|---------|
| 1. BUGS | Logic errors | Mutable default arguments, loop variable closures, unreachable code |
| 2. SECURITY | Vulnerabilities | eval/exec, pickle, subprocess shell=True, SQL injection, unsafe YAML, XML attacks |
| 3. TYPE SAFETY | Type correctness | Missing annotations, inconsistent returns, overly broad `Any` |
| 4. ASYNC PATTERNS | asyncio/threading | Blocking in async, missing await, sync sleep, thread safety |
| 5. API DESIGN | Django/Flask/FastAPI | Missing auth decorators, debug mode, insecure uploads, rate limiting |
| 6. PERFORMANCE | Efficiency | Generator vs list comprehension, quadratic string concat, inefficient loops |
| 7. CODE QUALITY | Pythonic | Bare except, mutable defaults, unused imports, missing context managers |

**Context-aware**: Django, Flask, FastAPI, SQLAlchemy, Pydantic, pytest, mypy/pyright.

| Command | Scope |
|---------|-------|
| `/enpitech:cr-python` | PR diff review |
| `/enpitech:cra-python` | Full codebase audit + system checks + dep audit |

---

## CR-General — Any Language

5 sequential **language-agnostic** review passes for platforms without a dedicated skill. Works with Vue.js, Angular, Svelte, Go, Rust, Ruby, PHP, Java, Kotlin, Swift, C#, and more.

| Pass | Focus | Examples |
|------|-------|---------|
| 1. BUGS | Logic errors | Null access, resource leaks, concurrency issues, off-by-one |
| 2. SECURITY | Vulnerabilities | Injection, hardcoded secrets, insecure crypto, SSRF, XSS, CSRF |
| 3. ERROR HANDLING | Resilience | Silent failures, bare exception catching, missing cleanup |
| 4. PERFORMANCE | Efficiency | Blocking I/O, N+1 queries, quadratic algorithms, memory issues |
| 5. CODE QUALITY | Maintainability | Dead code, duplication, overly complex functions, deprecated APIs |

**Auto-detects** language and framework from config files, then applies framework-specific checks (Vue.js `v-html` XSS, Go unchecked errors, Rails mass assignment, Laravel raw queries, Spring injection, etc.).

| Command | Scope |
|---------|-------|
| `/enpitech:cr-general` | PR diff review |
| `/enpitech:cra-general` | Full codebase audit + system checks + dep audit |

---

## CR-Deps — Dependency Audit

6 audit passes on dependency health. Use standalone — also included automatically in all `cra-*` audits.

| Pass | Focus | What It Checks |
|------|-------|----------------|
| 1. VULNERABILITIES | Security | `npm audit` / `pip-audit` — CVEs by severity |
| 2. OUTDATED | Freshness | Major version lag, security-related updates |
| 3. DEPRECATIONS | Lifecycle | Deprecated packages, suggested replacements |
| 4. LICENSE | Compliance | Copyleft in permissive projects, missing licenses |
| 5. UNUSED | Bloat | Declared but never imported dependencies |
| 6. LOCKFILE | Integrity | Missing lockfile, unpinned versions, sync issues |

Auto-detects npm, yarn, pnpm, pip, poetry, uv, pipenv.

```
/enpitech:cr-deps
```

> **Note:** Dependency audit does not run in `cr-*` diff reviews. It runs standalone via `cr-deps`, or automatically as part of any `cra-*` full audit.

---

## CR-Fullstack — Cross-Layer Review (Auto-Detect)

**Dynamically detects** which languages are in the project and applies the right criteria per layer. No hardcoded language combos.

**How detection works:**
- React/Next.js detected → applies `rules/react.md`
- Express/Fastify/Koa detected → applies `rules/node.md`
- Django/Flask/FastAPI detected → applies `rules/python.md`
- Vue, Angular, Svelte, Go, Rust, Ruby, PHP, Java, etc. → applies `rules/general.md`
- Monorepo → reads each workspace's config to classify

**Examples:**
- React + Express project → React passes on frontend, Node passes on backend
- Vue + Go project → General passes on both (auto-adapted to each)
- React + FastAPI project → React passes on frontend, Python passes on backend
- Angular + Spring Boot project → General passes on both

**Cross-layer checks** (always applied):
1. API contract validation
2. Shared type drift
3. Environment variable hygiene
4. Authentication flow consistency
5. Error contract matching
6. Data flow security
7. API versioning & deprecation

| Command | Scope |
|---------|-------|
| `/enpitech:cr-fullstack` | PR diff + cross-layer checks |
| `/enpitech:cra-fullstack` | Full audit per layer + cross-layer + system checks + dep audit |

---

## Installation

### Option A: Install as a plugin

**From a marketplace** (if your team has a [plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces) that includes this plugin):

```
/plugins install enpitech
```

**From a local directory:**

```bash
git clone https://github.com/enpitech/react-review.git
claude --plugin-dir ./react-review
```

Or add it permanently to your project's `.claude/plugins.json`.

Skills become `/enpitech:cr-react`, `/enpitech:cra-react`, `/enpitech:cr-node`, etc.

> **Note:** Plugins don't install workflow files. Copy the CI workflow manually:
> ```bash
> cp -r react-review/.github your-project/
> ```

### Option B: Copy files into your project

```bash
# Clone this repo
git clone https://github.com/enpitech/react-review.git

# Copy into your project
cp -r react-review/rules your-project/.claude/rules
cp -r react-review/skills your-project/.claude/skills
cp -r react-review/.github your-project/
```

Skills become `/cr-react`, `/cra-react`, `/cr-node`, etc. (no namespace prefix).

### Cherry-pick what you need

**Rules** (review criteria — pick by language):

| File | Purpose |
|------|---------|
| `rules/react.md` | React/Next.js 7-pass review criteria |
| `rules/node.md` | Node.js 7-pass review criteria |
| `rules/python.md` | Python 7-pass review criteria |
| `rules/general.md` | Language-agnostic 5-pass review criteria |
| `rules/deps.md` | Dependency audit criteria |
| `rules/fullstack.md` | Cross-layer check criteria |

**Skills** (pick by language + scope):

| File | Skill |
|------|-------|
| `skills/cr-react/SKILL.md` | PR diff React review |
| `skills/cra-react/SKILL.md` | Full codebase React audit |
| `skills/cr-node/SKILL.md` | PR diff Node.js review |
| `skills/cra-node/SKILL.md` | Full codebase Node.js audit |
| `skills/cr-python/SKILL.md` | PR diff Python review |
| `skills/cra-python/SKILL.md` | Full codebase Python audit |
| `skills/cr-general/SKILL.md` | PR diff general review |
| `skills/cra-general/SKILL.md` | Full codebase general audit |
| `skills/cr-deps/SKILL.md` | Dependency health audit |
| `skills/cr-fullstack/SKILL.md` | PR diff fullstack review |
| `skills/cra-fullstack/SKILL.md` | Full codebase fullstack audit |

**CI:**

| File | Purpose |
|------|---------|
| `.github/workflows/claude-code-review.yml` | CI workflow for all `cr-*` triggers |

## CI Setup

1. Copy `.github/workflows/claude-code-review.yml` into your repo
2. Add `ANTHROPIC_API_KEY` to your repo secrets (Settings → Secrets → Actions)
3. Comment one of these on any PR:

| Trigger | Review Type |
|---------|-------------|
| `/cr-general` | Language-agnostic code review |
| `/cr-fullstack` | Fullstack auto-detect + cross-layer |
| `/cr-react` | React/Next.js code review |
| `/cr-node` | Node.js code review |
| `/cr-python` | Python code review |
| `/cr-deps` | Dependency health audit |
| `/claude-review` | React review (backward compatible) |

> **Note:** `cra-*` skills (full audits) are local-only and not triggered in CI.

The workflow:
- Detects the trigger keyword and selects the appropriate review criteria
- Only runs for repo collaborators (OWNER/MEMBER/COLLABORATOR)
- Posts an acknowledgment comment, then inline review comments
- Claude cannot post arbitrary PR comments (hardened against prompt injection)

### Optional: Auto-trigger on PR

Uncomment the `pull_request` trigger in the workflow to run automatically on every PR:

```yaml
on:
  pull_request:
    types: [opened, synchronize]
  issue_comment:
    types: [created]
```

## Security

- `gh pr comment` removed from Claude's allowed tools — prevents prompt injection via malicious PR descriptions
- Claude can only post structured inline comments via MCP tool
- Completion message is posted by the workflow itself, outside Claude's control
- Trigger restricted to repo OWNER/MEMBER/COLLABORATOR only

## Customization

Edit the criteria files in `rules/` to:
- Add/remove review passes
- Adjust confidence threshold (default: 8/10)
- Change severity levels
- Add framework-specific checks
- Modify context-aware detection rules

| File | Controls |
|------|----------|
| `rules/react.md` | React/Next.js review rules |
| `rules/node.md` | Node.js review rules |
| `rules/python.md` | Python review rules |
| `rules/general.md` | Language-agnostic review rules |
| `rules/deps.md` | Dependency audit rules |
| `rules/fullstack.md` | Cross-layer check rules |

All skills reference these files — single source of truth per concern.

## License

MIT

## License

MIT
