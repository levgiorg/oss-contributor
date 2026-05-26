# Issue Scout Skill

Find viable issues for contribution. Filter out claimed, duplicated, or blocked issues.

## Issue Discovery

```bash
# Good issues
gh issue list -R <owner>/<repo> --label "good first issue" --state open --limit 50
gh issue list -R <owner>/<repo> --label "help wanted" --state open --limit 50
gh issue list -R <owner>/<repo> --label "bug" --state open --limit 50

# For repos without labels (opencode), fetch all open:
gh issue list -R <owner>/<repo> --state open --limit 100
```

## Triple Deduplication Check (MANDATORY before working)

### CHECK 1: Existing open PRs
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

### CHECK 3: Recent ownership claims
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
- Missing handler/registration (e.g., unregistered tool, missing keybind)

### ALWAYS SKIP:
- Feature requests (need design review first)
- Issues touching server API or SDK (needs regeneration)
- Issues opened < 3 days ago
- "needs:compliance" label (AI-generated, auto-closes)
- Fixes touching > 5 files
- Issues labeled "wontfix", "invalid", "duplicate", "on hold", "design needed"
- Billing/support questions
- Issues with 5+ people already requesting assignment

## Repo-specific Rules

### anomalyco/opencode (Bun/TypeScript)
- Base branch: **dev** (not main)
- Runtime: Bun
- Must use PR template exactly
- `needs:compliance` issues auto-close in 2 hours
- Must comment on issue before PR (optional but recommended)
- CSRs (Closed-Source Requests) are different from regular issues

### huggingface/smolagents (Python)
- Base branch: **main**
- Uses ruff + pytest
- Uses `make quality` / `make style` / `make test`
- First-time contributor CI may not run automatically

### langchain-ai/langgraph (Python)
- Base branch: **main**
- Has **auto-close bot** — PRs from unassigned contributors are closed
- MUST comment requesting assignment before opening PR
- Write "I have a fix ready in PR #XXXX that was auto-closed" after it closes
- CLA may be required

### langchain-ai/langchain (Python)
- Base branch: **master** (not main)
- Has **auto-close bot** — same as langgraph
- CLA required
- EXTREMELY competitive — most issues have 5+ people competing
- Only feasible for very simple fixes

### pydantic/pydantic-ai (Python)
- **Requires explicit maintainer assignment before PR**
- "Unassigned PRs may be auto-closed"
- Cannot contribute autonomously — must wait for assignment
- Features go to pydantic-ai-harness, not core

## Priority Order
1. Bug with reproduction script → easiest to verify
2. Missing test / test gap → always welcome
3. Small improvement with clear spec → well-defined
4. Type hint / error handling → safe changes
