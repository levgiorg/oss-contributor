# PR Crafter Skill

Create well-formed pull requests that pass CI and don't get auto-closed.

## STEP 0 — READ THE ACTUAL PR TEMPLATE (DO NOT SKIP)

**Before you open any PR, read the repo's ACTUAL template file:**
```bash
cat .github/pull_request_template.md 2>/dev/null
cat .github/PULL_REQUEST_TEMPLATE.md 2>/dev/null
cat CONTRIBUTING.md 2>/dev/null
cat .github/CONTRIBUTING.md 2>/dev/null
```

**Every repo has its own template. Copy it exactly.** The templates below are just examples — the repo's actual template file takes precedence.

**Why this matters:**
- opencode's compliance bot checks for EXACT section headers from `.github/pull_request_template.md`
- Missing a single section (even "Screenshots / recordings" for non-UI changes) triggers `needs:compliance` → auto-close in 2h
- LangChain repos check for `Closes #N` and issue assignment
- Some repos have custom CI checks that parse the PR body for specific fields

## Branch Naming
```
fix/issue-<N>-<short-slug>
```
Keep slug short (2-4 words), kebab-case.

## Commit Format
```
fix(<scope>): <imperative description, ≤72 chars>

<2-3 sentences: what was broken, what you changed, why correct.>

Fixes #<N>
```
- Conventional commit: fix:, feat:, test:, refactor:, docs:, chore:
- Subject in imperative mood: "fix" not "fixed" or "fixes"
- `Fixes #N` on its own last line triggers auto-close on merge
- No emojis, no `# TODO`, no AI-sounding language

## Cross-Fork PR Creation

**This is the most common failure point.** When you clone your fork, `gh pr create` defaults to opening a PR against your fork. You MUST specify the upstream repo:

```bash
gh pr create \
  --repo <UPSTREAM_OWNER>/<UPSTREAM_REPO> \
  --head levgiorg:<branch-name> \
  --base <dev|main|master> \
  --title "<conventional-commit-title>" \
  --body-file /tmp/pr_body.md
```

### Correct base branches:
| Repo | Base |
|------|------|
| anomalyco/opencode | **dev** |
| huggingface/smolagents | **main** |
| langchain-ai/langgraph | **main** |
| langchain-ai/langchain | **master** |
| pydantic/pydantic-ai | **main** |

### If you accidentally open a PR against levgiorg/<repo>:
1. `gh pr close <N> -R levgiorg/<repo>`
2. Re-push the branch if needed
3. Re-create with `--repo <UPSTREAM>` flag

## PR Body Template: anomalyco/opencode

MUST match this exact template. The compliance bot auto-closes incomplete templates.

```
### Issue for this PR

Closes #<N>

### Type of change

- [x] Bug fix

### What does this PR do?

<1-2 sentences describing the fix clearly. No walls of text.>

### How did you verify your code works?

- [x] TypeScript typecheck passes
- [x] Existing tests pass

### Screenshots / recordings

_Not applicable — no UI change._

### Checklist

- [x] I have tested my changes locally
- [x] I have not included unrelated changes in this PR
```

For UI changes, add screenshots under "Screenshots / recordings".

## PR Body Template: Python repos (smolagents, langgraph, langchain)

```
## Summary

<2-3 sentences: what was broken, what you changed, why correct>

## Root Cause

Exact file/function/line responsible: `<file>:<line>` in `<function>()`

## Changes

- `<file>`: <what was changed and why>

## Testing

- [x] All tests pass: `pytest tests/ -x -q`
- [x] Linter clean: `ruff check` / `ruff format`
- [x] Type checker clean (if applicable)

## Related Issue

Closes #<N>
```

## Anti-patterns

1. **AI-generated walls of text** → PR gets ignored or closed. Keep descriptions short and direct.
2. **Missing "Closes #N"** → won't auto-close issue on merge
3. **Wrong base branch** → PR targets wrong base
4. **PR against fork** → no one sees it, wasted effort
5. **Huge diffs** → `bun install` lockfile changes, generated SDK files. Discard before committing.
6. **No verification** → PR body must explain how you tested
7. **Unrelated changes** → CI bots flag this, PR gets closed

## CI Verification

### Before opening:
```bash
# TypeScript/Bun repos
export PATH="$HOME/.bun/bin:$PATH"
cd packages/<pkg> && bun run typecheck  # MUST pass
cd packages/<pkg> && bun test           # MUST pass

# Python repos
ruff format . --check   # MUST pass
ruff check .            # MUST pass
pytest tests/ -x -q     # MUST pass
```

### After opening:
```bash
gh pr checks <pr_number> -R <owner>/<repo> --watch
```

## Handling Auto-Close Bots

LangChain/langgraph repos auto-close PRs from unassigned contributors. Strategy:

1. Comment on the issue: "Hi! I'd like to work on this. Could a maintainer assign me?"
2. Open the PR anyway (it will get closed)
3. After it closes, comment again: "My PR #XXXX was auto-closed. I have the fix ready. Could a maintainer reopen or assign me?"
4. The PR stays visible to maintainers and can be reopened with one click

This is the best you can do autonomously. Maintainers control assignment.
