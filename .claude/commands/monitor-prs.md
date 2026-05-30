# Monitor PRs

Check all open PRs for new comments, labels, and reviews. Act on issues automatically. Designed to run on a loop.

## How to Start

**Session-bound (runs while Claude Code is open):**
```
/loop 60m /monitor-prs
```

**Unattended (survives session close):**
Use `/schedule` and describe: "Every 60 minutes, run /monitor-prs in /Users/georgelevis/Projects/Contibuting"

---

## Monitoring Procedure

### Step 1 — Load open PRs from contribution_log.json

```bash
cat contribution_log.json | \
  python3 -c "
import json, sys
log = json.load(sys.stdin)
open_prs = [e for e in log if not e.get('skipped') or e.get('ci_status') == 'closed']
for pr in open_prs:
    print(f\"{pr['repo']}#{pr['pr_number']} — {pr['issue_title'][:60]}\")
"
```

### Step 2 — For each PR, gather live state

```bash
# Comments (bots + maintainers)
gh pr view <pr_number> -R <owner>/<repo> --json comments \
  --jq '.comments[] | "\(.author.login) [\(.createdAt)]: \(.body[:300])"'

# Labels
gh pr view <pr_number> -R <owner>/<repo> --json labels \
  --jq '[.labels[].name]'

# Maintainer reviews
gh pr view <pr_number> -R <owner>/<repo> --json reviews \
  --jq '.reviews[] | "\(.author.login): \(.state) — \(.body[:200])"'

# CI status
gh pr checks <pr_number> -R <owner>/<repo> \
  --json name,conclusion --jq '.[] | "\(.name): \(.conclusion)"'

# Overall PR state (open/closed/merged)
gh pr view <pr_number> -R <owner>/<repo> --json state,mergedAt \
  --jq '"state=\(.state) merged=\(.mergedAt)"'
```

### Step 3 — Act on findings

#### ACTION: needs:compliance label detected
Auto-fix immediately — do not wait.

```bash
# For anomalyco/opencode PRs
gh api repos/anomalyco/opencode/pulls/<N> --method PATCH \
  -f body='### Issue for this PR
Closes #<issue_number>

### Type of change
- [x] Bug fix

### What does this PR do?
<fetch from existing PR description or issue title>

### How did you verify your code works?
- [x] TypeScript typecheck passes
- [x] Existing tests pass

### Screenshots / recordings
_Not applicable — no UI change._

### Checklist
- [x] I have tested my changes locally
- [x] I have not included unrelated changes in this PR'
```

Then verify the label is removed:
```bash
gh pr view <N> -R anomalyco/opencode --json labels --jq '[.labels[].name]'
```

#### ACTION: PR auto-closed by bot (langgraph/langchain)
Comment on the parent ISSUE (not the PR) referencing the closed PR:

```bash
gh issue comment <issue_number> -R <owner>/<repo> \
  --body "My PR #<pr_number> was auto-closed because I'm not assigned to this issue. The fix is ready. Could a maintainer please assign me or reopen the PR? (PR: <pr_url>)"
```

Then update `contribution_log.json`:
```bash
# Set ci_status to "closed" and add skip_reason
python3 -c "
import json
with open('contribution_log.json') as f:
    log = json.load(f)
for entry in log:
    if entry['pr_number'] == <N>:
        entry['ci_status'] = 'closed'
        entry['skipped'] = True
        entry['skip_reason'] = 'auto-closed: contributor not assigned to issue. Commented requesting assignment.'
with open('contribution_log.json', 'w') as f:
    json.dump(log, f, indent=2)
"
```

#### ACTION: PR merged
Celebrate and update log:
```bash
python3 -c "
import json
with open('contribution_log.json') as f:
    log = json.load(f)
for entry in log:
    if entry['pr_number'] == <N>:
        entry['ci_status'] = 'merged'
with open('contribution_log.json', 'w') as f:
    json.dump(log, f, indent=2)
"
```

#### ACTION: Maintainer review requesting changes

**Evaluate the review comment:** Is this a simple, scoped change request?

**Auto-fix if the request is clear and scoped** (examples of auto-fixable comments):
- "This PR seems to be missing the part that prints out the error"
- "Could you add a test for this edge case?"
- "The variable name should be X instead of Y"
- "Please add a comment explaining why this is needed"
- "This should use `logError` instead of `console.log`"

**How to auto-fix:**
1. Clone the branch if not already cloned:
   ```bash
   gh repo clone levgiorg/<repo> /tmp/<repo>-<pr_number>
   cd /tmp/<repo>-<pr_number>
   git fetch origin <branch_name>
   git checkout <branch_name>
   ```
2. Make the requested change (minimal, scoped to the review)
3. Run CI locally:
   ```bash
   # TypeScript: export PATH="$HOME/.bun/bin:$PATH"; bun run typecheck && bun test
   # Python: ruff check . && pytest tests/ -x -q
   ```
4. Commit with conventional format:
   ```
   fix: <address review feedback>
   ```
5. Push and reply to the review comment:
   ```bash
   gh pr comment <N> -R <owner>/<repo> --body "Good catch — fixed in <commit_hash>. Thanks!"
   ```

**Surface for human review ONLY when the request is ambiguous:**
- "This approach won't work, please redesign"
- "Can we discuss the architecture of this?"
- "Have you considered using pattern X instead?" (when unclear preference)
- Any comment asking for a design-level change

When surfacing, format clearly:
```
PR #<N> (<repo>): Maintainer <username> requested changes:
  "<review comment>"
  → Branch: <branch>
  → File: check the PR for specific file/line references
  ACTION NEEDED: Review the feedback and decide how to proceed.
```

#### ACTION: Potential duplicate comment
Check the flagged PR:
```bash
gh pr view <flagged_pr_number> -R <owner>/<repo> --json title,additions,deletions,files \
  --jq '"Title: \(.title) | +\(.additions)/-\(.deletions) lines"'
```
- If truly identical: `gh pr close <our_pr_number> -R <owner>/<repo> --comment "Closing in favour of #<flagged_pr> which covers the same fix."`
- If different approach: add a comment to our PR explaining the distinction

#### ACTION: All CI green, no labels, no review requests
```
PR #<N> (<repo>): All checks green. No action needed.
```

### Step 4 — Emit one-line summary per PR

After processing all PRs, print a status table:

```
=== PR Monitor — <timestamp> ===
anomalyco/opencode#29370  — PENDING  — no new activity
anomalyco/opencode#29371  — PENDING  — no new activity
anomalyco/opencode#29378  — FIXED    — compliance label cleared
smolagents#2311           — PENDING  — CI not yet triggered (first-time contributor)
langgraph#7909            — CLOSED   — commented on issue #7684 requesting assignment
```

## What NOT to Auto-Fix

- Maintainer code review feedback → surface it, let a human decide
- "This is a duplicate of #X" when the diffs are genuinely different → surface it
- Any PR where `ci_status == "merged"` → skip entirely

## Keeping contribution_log.json Accurate

After each monitor run, ensure every entry reflects actual live state. The log is the single source of truth for `/oss-contribute` and `/issue-scout` dedup checks.
