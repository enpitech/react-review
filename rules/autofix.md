# Autofix Workflow

Shared autofix behavior for all review and audit skills. Referenced from the final step of each skill.

## Environment Detection

Detect the execution environment before choosing a mode:
- **CI mode**: `CI=true` environment variable is set (standard in GitHub Actions, GitLab CI, etc.)
- **Local mode**: `CI` is not set or is `false`

---

## Local Mode

After completing all review passes, show the findings summary to the user and present these options:

> **What would you like to do next?**
> 1. **Create findings file** — save all findings to the findings `.md` file
> 2. **Fix step by step** — review and approve each fix individually
> 3. **Fix all** — apply all suggested fixes at once

### Option 1 — Create findings file
Write all findings to the skill's findings `.md` file in the project root (e.g., `cr-react-findings.md`). Confirm the file was created and show its path.

### Option 2 — Fix step by step
For each finding, in severity order (CRITICAL first, then WARNING):
1. Show the finding: file, line, issue description, and the full proposed code change
2. Ask: **"Apply this fix? (yes / skip)"**
3. If **yes** — apply the fix and confirm it was applied
4. If **skip** — move to the next finding without changes
5. After all findings are reviewed, show a summary:
   - Total reviewed: N
   - Applied: X
   - Skipped: Y

### Option 3 — Fix all
Apply all suggested fixes automatically in severity order (CRITICAL first, then WARNING).
Show a summary when complete with the list of all changes made.

---

## CI Mode (CR skills only — not applicable to CRA skills)

When running in CI, do **not** prompt interactively. Instead, post findings as GitHub PR comments using `gh` CLI.

### Step 1 — Post main review comment

Create a new PR comment thread summarizing all findings:

````
## Code Review — {total_count} issues found

| Severity | Count |
|----------|-------|
| CRITICAL | {critical_count} |
| WARNING  | {warning_count} |

{brief list of all findings with file:line references}

> To fix all issues below, reply with `/fix-all`
````

Use `gh pr comment <number> --body "<body>"` to post the main comment. Store the comment URL for threading.

### Step 2 — Post individual finding replies

For each finding, post a **reply** in the same thread as the main comment:

````
### [{SEVERITY}] {file}:{line}

**Issue:** {description}
**Why:** {reason}

**Suggested fix:**
```suggestion
{code change}
```

> To fix this issue, reply with `/fix`
````

Use `gh pr comment <number> --body "<body>"` for each finding reply.

### Fix command behavior

When the CI workflow detects a `/fix` or `/fix-all` reply and re-invokes the agent:
- **`/fix`** on an individual finding comment — apply the fix for that specific finding only, commit, and push
- **`/fix-all`** on the main review comment — apply all suggested fixes, commit, and push
