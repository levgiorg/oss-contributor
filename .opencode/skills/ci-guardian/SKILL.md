# CI Guardian Skill

Ensure all CI checks pass before and after opening PRs.

## Pre-Push Checklist

### TypeScript/Bun repos (opencode)
```bash
export PATH="$HOME/.bun/bin:$PATH"

# 1. TypeScript typecheck (MUST pass)
cd packages/opencode && bun run typecheck
# OR from root:
bun run typecheck

# 2. Unit tests (MUST pass)
cd packages/opencode && bun test

# 3. Lint (informational — format issues only)
bun run lint 2>/dev/null || true
```

### Python repos (smolagents, langgraph, langchain)
```bash
# 1. Formatter (MUST pass)
ruff format . --check

# 2. Linter (MUST pass)
ruff check .

# 3. Tests (MUST pass)
pytest tests/ -x -q --tb=short

# 4. Makefile targets if available
make lint 2>/dev/null || true
make test 2>/dev/null || true
```

## Common CI Failures & Fixes

### anomalyco/opencode

| Check | What it does | Common failure |
|-------|-------------|----------------|
| `check-duplicates` | Detects duplicate PRs | Your fix matches an existing PR's diff |
| `check-standards` | AGENTS.md style enforcement | Deep nesting, `any` types, unnecessary destructuring |
| `add-contributor-label` | Adds contributor label | Never fails |
| `check-compliance` | PR template validation | Missing template sections, empty fields |

**Fix for `check-compliance`**: Update PR body to include ALL template sections:
```bash
gh api repos/anomalyco/opencode/pulls/<N> --method PATCH \
  -f body='### Issue for this PR
...full template...
### Checklist
- [x] I have tested my changes locally
- [x] I have not included unrelated changes in this PR'
```

**Fix for duplicate detection**: Check the flagged PR. If it's a true duplicate and the other PR is better, close yours. If your approach is different, add a comment explaining why.

### langchain-ai/langgraph

| Check | What it does | Common failure |
|-------|-------------|----------------|
| `check-issue-link` | Verifies contributor is assigned to linked issue | NOT ASSIGNED → all checks fail |
| `CI Success` | Meta check | Fails if any other check fails |
| `cd libs/*/test` | Per-package tests | Cascading failure from check-issue-link |
| `cd libs/*/lint` | Per-package lint | Cascading failure from check-issue-link |

**The auto-close bot is the root cause.** All failures cascade from `check-issue-link`. You cannot fix this without maintainer assignment.

### huggingface/smolagents

- CI may not run automatically for first-time contributors
- Maintainer approval needed to trigger CI
- Local verification (ruff + pytest) is the best you can do

## Post-PR Monitoring (CRITICAL — DO NOT SKIP)

After opening a PR, **immediately** monitor it for issues. Bots and maintainers will comment within minutes.

### 1. Monitor CI
```bash
# Watch all CI checks complete
gh pr checks <pr_number> -R <owner>/<repo> --watch --interval 30
```

### 2. Check for bot comments (ALWAYS do this)
```bash
# See ALL comments on your PR
gh pr view <pr_number> -R <owner>/<repo> --json comments \
  --jq '.comments[] | "\(.author.login) | \(.createdAt): \(.body[:200])"'

# Check specifically for bot comments (compliance, duplicates, standards)
gh pr view <pr_number> -R <owner>/<repo> --json comments \
  --jq '.comments[] | select(.author.login == "github-actions") | .body'
```

### 3. Check for maintainer reviews
```bash
gh pr view <pr_number> -R <owner>/<repo> --json reviews \
  --jq '.reviews[] | "\(.author.login): \(.state) — \(.body[:200])"'
```

### 4. Check labels (compliance/auto-close labels!)
```bash
gh pr view <pr_number> -R <owner>/<repo> --json labels \
  --jq '.labels[] | .name'
```

### 5. Respond to issues IMMEDIATELY

| What you see | Action |
|-------------|--------|
| `needs:compliance` label | Update PR body to match template EXACTLY via `gh api repos/.../pulls/<N> --method PATCH -f body='...'` |
| "Potential duplicate" comment | Check the flagged PR. If identical, close yours. If different approach, comment why. |
| Maintainer review requesting changes | Push the fix. Do NOT argue — just fix it. |
| Auto-close bot comment | Comment on the ISSUE (not PR) requesting assignment. Reference your closed PR number. |
| All CI green, no comments | Great — now wait. Do NOT ping maintainers. |

### 6. Monitor ALL your open PRs at once
```bash
# List all open PRs with their CI status
gh search prs --author levgiorg --state open --json number,title,repository,statusCheckRollup,labels \
  --jq '.[] | "\(.repository.name)#\(.number): \([.labels[].name]?) — checks: \([.statusCheckRollup[]?.conclusion] | unique)"'
```

### Common post-PR scenarios from real experience:

**opencode #29378 — compliance bot auto-close threat:**
- Bot commented: "PR description is missing required template sections"
- Label `needs:compliance` applied → auto-close in 2 hours if not fixed
- Fix: Updated PR body via `gh api PATCH` with ALL template sections
- Bot re-checked and commented: "Thanks for updating your PR! It now meets our contributing guidelines."

**opencode #29370 — duplicate detection:**
- Bot commented: "Potential Duplicate Found: PR #29276"
- Both PRs had identical diffs (same fix for /compact command)
- No auto-close, but maintainer will likely pick the first-opened PR

**opencode #29371 — dirty diff (300+ files):**
- Force-pushed clean branch from fresh clone
- Diff went from 300+ files to 1 file / 1 line
- CI re-ran with only the relevant change

**langgraph #7909, #7911, #7912 — auto-close bot:**
- All 3 PRs auto-closed within seconds of opening
- Commented on each issue referencing the closed PR number
- Maintainer must assign before PRs reopen

## Husky Pre-Push Hook (opencode)

The opencode repo has a husky pre-push hook that runs `bun turbo typecheck`. If `bun` is not in PATH:

```bash
# Fix: add bun to PATH
export PATH="$HOME/.bun/bin:$PATH"
git push origin <branch>
```

## Dirty Diff Prevention

After `bun install`, the working tree gets polluted with:
- `bun.lock` changes
- Generated SDK files (from `./script/generate.ts`)
- Untracked build artifacts

**Prevention**:
```bash
# After bun install, immediately discard non-source changes:
git checkout -- bun.lock 2>/dev/null
git checkout -- packages/ 2>/dev/null  
git clean -fd 2>/dev/null

# Only stage your source changes:
git add packages/opencode/src/<file-you-changed>.ts
```

**Recovery** (if you already committed dirty files):
```bash
# Rebase to remove the dirty commit
git reset --soft HEAD~1
git reset HEAD -- .
git add <only-your-source-files>
git commit -m "fix: your message"
git push --force
```
