# React Review

7-pass React/Next.js code review [plugin](https://code.claude.com/docs/en/plugins) for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) — CI + local [skills](https://code.claude.com/docs/en/skills).

## What It Does

Runs 7 sequential review passes on your React/Next.js code, catching real bugs instead of style nits:

| Pass | Focus | Examples |
|------|-------|---------|
| 1. BUGS | Logic errors | null access, race conditions, wrong conditionals |
| 2. SECURITY | Vulnerabilities | XSS, exposed secrets, unauthenticated Server Actions |
| 3. COMPONENT ARCHITECTURE | Structure | God components, prop drilling, cross-feature imports |
| 4. HOOKS & STATE | React patterns | Derived state in useEffect, stale closures, joinable hooks |
| 5. PERFORMANCE | Speed | Sequential awaits, missing dynamic imports, barrel file imports |
| 6. CODE QUALITY | React-specific | Array mutation, missing error boundaries, duplicated logic |
| 7. INTENT CHECK | PR scope | Unrelated changes that snuck into the diff |

Only reports **CRITICAL** (will break in prod) and **WARNING** (likely bug/security risk) at **8/10+ confidence**. No noise.

### Context-Aware

Automatically adjusts based on your `package.json`:
- **React Compiler** detected → skips unnecessary memoization findings
- **Next.js** detected → enables SSR/RSC rules (Server Action auth, serialization, Suspense)
- **Design system** detected (`@radix-ui`, `shadcn`) → flags excessive inline styling

## Two Skills

### `/enpitech:pr-review` — PR Review (CI + Local)

Scoped to the PR diff + directly affected files. Fast and focused.

**CI — triggered by `/claude-review` comment on any PR:**

Posts inline comments directly on the changed lines.

**Local — via Claude Code CLI:**

```
/enpitech:pr-review
```

Outputs findings to `pr-review-findings.md` and offers to apply fixes.

### `/enpitech:sys-review` — System Review (Local Only)

Full codebase audit. Runs all 7 passes across every source file, plus system-level checks:

```
/enpitech:sys-review
```

Catches things a PR review never could:
- Dead exports, circular dependencies
- Duplicated patterns across distant files
- Architecture drift, inconsistent state patterns
- Bundle size hotspots

Outputs organized report to `system-review-findings.md`.

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

Skills become `/enpitech:pr-review` and `/enpitech:sys-review`.

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

Skills become `/pr-review` and `/sys-review` (no namespace prefix).

### Cherry-pick what you need

| File | Purpose | Required? |
|------|---------|-----------|
| `rules/pr-review-criteria.md` | Shared 7-pass review criteria | Yes |
| `skills/pr-review/SKILL.md` | `/enpitech:pr-review` skill (or `/pr-review` standalone) | For local use |
| `skills/sys-review/SKILL.md` | `/enpitech:sys-review` skill (or `/sys-review` standalone) | For local use |
| `.github/workflows/claude-code-review.yml` | CI workflow | For GitHub Actions |

## CI Setup

1. Copy `.github/workflows/claude-code-review.yml` into your repo
2. Add `ANTHROPIC_API_KEY` to your repo secrets (Settings → Secrets → Actions)
3. Comment `/claude-review` on any PR

The workflow:
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

Edit `rules/pr-review-criteria.md` to:
- Add/remove review passes
- Adjust confidence threshold (default: 8/10)
- Change severity levels
- Add framework-specific checks
- Modify context-aware detection rules

Both skills and the CI workflow reference this single file — one source of truth.

## License

MIT
