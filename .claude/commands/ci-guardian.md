# CI Guardian

Ensure all CI checks pass before and after opening PRs. Monitor and fix post-PR issues.

## Pre-Push Verification

### TypeScript/Bun repos (opencode)
```bash
export PATH="$HOME/.bun/bin:$PATH"

# MUST pass — do not push if these fail
cd packages/opencode && bun run typecheck
cd packages/opencode && bun test
```

### Python repos (smolagents, langgraph, langchain)
```bash
# MUST pass
ruff format . --check
ruff check .
pytest tests/ -x -q --tb=short

# If Makefile available
make lint 2>/dev/null || true
make test 2>/dev/null || true
```

## Common CI Failures & Fixes

### anomalyco/opencode

| Check | Failure | Fix |
|-------|---------|-----|
| `check-compliance` | Missing PR template sections | Update PR body via `gh api PATCH` with all sections |
| `check-duplicates` | Your diff matches existing PR | Check flagged PR; close yours if identical |
| `check-standards` | Deep nesting, `any` types | Fix code style per AGENTS.md in that repo |
| `add-contributor-label` | Never fails | — |

**Fix compliance label immediately:**
```bash
gh api repos/anomalyco/opencode/pulls/<N> --method PATCH \
  -f body='### Issue for this PR
Closes #<N>

### Type of change
- [x] Bug fix

### What does this PR do?
<description>

### How did you verify your code works?
- [x] TypeScript typecheck passes
- [x] Existing tests pass

### Screenshots / recordings
_Not applicable — no UI change._

### Checklist
- [x] I have tested my changes locally
- [x] I have not included unrelated changes in this PR'
```

### langchain-ai/langgraph

All CI failures cascade from `check-issue-link`. Root cause: not assigned to the issue.
Cannot fix without maintainer assignment — comment on the issue and wait.

### huggingface/smolagents

CI may not trigger for first-time contributors. Maintainer must approve the first run.
Local ruff + pytest verification is the best you can do.

## Post-PR Monitoring

### Monitor CI
```bash
gh pr checks <pr_number> -R <owner>/<repo> --watch --interval 30
```

### Check bot comments
```bash
gh pr view <pr_number> -R <owner>/<repo> --json comments \
  --jq '.comments[] | "\(.author.login) | \(.createdAt): \(.body[:300])"'

# Only bot comments:
gh pr view <pr_number> -R <owner>/<repo> --json comments \
  --jq '.comments[] | select(.author.login == "github-actions") | .body'
```

### Check maintainer reviews
```bash
gh pr view <pr_number> -R <owner>/<repo> --json reviews \
  --jq '.reviews[] | "\(.author.login): \(.state) — \(.body[:200])"'
```

### Check labels
```bash
gh pr view <pr_number> -R <owner>/<repo> --json labels \
  --jq '.labels[].name'
```

### Response protocol

| What you see | Action |
|-------------|--------|
| `needs:compliance` label | Update PR body immediately via `gh api PATCH` |
| "Potential duplicate" comment | Check the flagged PR. If identical → close yours. If different → comment why. |
| Maintainer review requesting changes | Push the fix. Never argue. |
| Auto-close bot comment | Comment on the ISSUE requesting assignment. Reference your closed PR number. |
| All CI green, no comments | Wait. Do NOT ping maintainers. |

### Monitor all open PRs at once
```bash
gh search prs --author levgiorg --state open \
  --json number,title,repository,statusCheckRollup,labels \
  --jq '.[] | "\(.repository.name)#\(.number): \([.labels[].name]?) — \([.statusCheckRollup[]?.conclusion] | unique)"'
```

## Husky Pre-Push Hook (opencode)

```bash
# Always add bun to PATH before pushing to opencode
export PATH="$HOME/.bun/bin:$PATH"
git push origin <branch>
```

## Dirty Diff Prevention

After `bun install`:
```bash
git checkout -- bun.lock 2>/dev/null
git checkout -- packages/ 2>/dev/null
git clean -fd 2>/dev/null
git add packages/opencode/src/<file-you-changed>.ts
```

If you already committed dirty files:
```bash
git reset --soft HEAD~1
git reset HEAD -- .
git add <only-your-source-files>
git commit -m "fix: your message"
git push --force
```
