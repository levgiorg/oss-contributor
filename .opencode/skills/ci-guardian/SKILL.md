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

## Post-Open CI Monitoring

```bash
# Watch CI for a specific PR
gh pr checks <pr_number> -R <owner>/<repo> --watch --interval 30

# Check failed CI logs
gh run view --log-failed

# Check PR for bot comments
gh pr view <pr_number> -R <owner>/<repo> --json comments \
  --jq '.comments[] | select(.author.login == "github-actions") | .body'
```

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
