# Issue Scout

Find viable issues for contribution. Filter out claimed, duplicated, or blocked issues.

## Issue Discovery

```bash
# Labeled issues
gh issue list -R <owner>/<repo> --label "good first issue" --state open --limit 50
gh issue list -R <owner>/<repo> --label "help wanted" --state open --limit 50
gh issue list -R <owner>/<repo> --label "bug" --state open --limit 50

# Repos without labels (opencode):
gh issue list -R <owner>/<repo> --state open --limit 100
```

## Triple Deduplication Check (MANDATORY before working on any issue)

### CHECK 1: Existing open PRs for this issue
```bash
gh pr list -R <owner>/<repo> --state open \
  --search "fixes #<N> OR closes #<N> OR resolves #<N>" \
  --json number,title,url
```
**SKIP if any open PR exists.**

### CHECK 2: Cross-referenced PRs
```bash
gh api "repos/<owner>/<repo>/issues/<N>/timeline" \
  --jq '.[] | select(.event == "cross-referenced") | .source.issue | {number, state, title}'
```
**SKIP if cross-referenced to an open PR.**

### CHECK 3: Recent ownership claims (last 7 days)
```bash
gh issue view <N> -R <owner>/<repo> --comments \
  --json comments --jq '.comments[].body' | \
  grep -i "working on\|pr #\|pull request\|opened a pr\|assigned" | head -5
```
**SKIP if anyone claimed it in the last 7 days.**

## Selection Rules

### ALWAYS PICK:
- Bug with clear reproduction steps
- Missing test for existing feature
- Small scoped improvement with clear spec
- Type hint / error handling improvement
- Missing handler/registration (unregistered tool, missing keybind)

### ALWAYS SKIP:
- Feature requests (need design review first)
- Issues touching server API or SDK (needs regeneration)
- Issues opened < 3 days ago
- `needs:compliance` label (AI-generated, auto-closes)
- Fixes touching > 5 files
- Labels: `wontfix`, `invalid`, `duplicate`, `on hold`, `design needed`
- Billing/support questions
- Issues with 5+ people already requesting assignment

## Repo-Specific Rules

### anomalyco/opencode (Bun/TypeScript)
- Base branch: **dev** (not main)
- PR template enforcement is STRICT — compliance bot auto-closes in 2h
- Comment on issue before PR (optional but recommended)

### huggingface/smolagents (Python)
- Base branch: **main**
- Uses `make quality` / `make style` / `make test`
- First-time contributor CI may not run automatically — needs maintainer approval

### langchain-ai/langgraph (Python)
- Base branch: **main**
- **Auto-close bot** — PRs from unassigned contributors are closed
- Comment requesting assignment BEFORE opening PR
- After auto-close, comment on issue: "My PR #XXXX was auto-closed. Fix is ready."

### langchain-ai/langchain (Python)
- Base branch: **master**
- Auto-close bot + CLA required
- Most issues have 5+ competitors — skip unless trivially simple

### pydantic/pydantic-ai (Python)
- Requires explicit maintainer assignment before any PR
- Cannot contribute autonomously — skip

## Priority Order
1. Bug with reproduction script
2. Missing test / test gap
3. Small improvement with clear spec
4. Type hint / error handling
