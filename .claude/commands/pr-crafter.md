# PR Crafter

Create well-formed pull requests that pass compliance bots and don't get auto-closed.

## STEP 0 — READ THE ACTUAL PR TEMPLATE (DO NOT SKIP)

```bash
cat .github/pull_request_template.md 2>/dev/null
cat .github/PULL_REQUEST_TEMPLATE.md 2>/dev/null
cat CONTRIBUTING.md 2>/dev/null
```

Copy the actual template into `/tmp/pr_body.md` before writing any code. Every repo differs. Templates below are defaults only — the repo's file takes precedence.

## Branch Naming
```
fix/issue-<N>-<short-slug>
```
Slug: 2-4 words, kebab-case.

## Commit Format
```
fix(<scope>): <imperative description ≤72 chars>

<2-3 sentences: what was broken, what you changed, why correct.>

Fixes #<N>
```
- Conventional commit format
- Imperative mood: "fix" not "fixed"
- `Fixes #N` on its own last line
- No emojis, no `# TODO`, no AI-sounding language

## Cross-Fork PR Creation

Most common failure point — always specify upstream explicitly:

```bash
gh pr create \
  --repo <UPSTREAM_OWNER>/<UPSTREAM_REPO> \
  --head levgiorg:<branch-name> \
  --base <dev|main|master> \
  --title "<conventional-commit-title>" \
  --body-file /tmp/pr_body.md
```

### Correct base branches
| Repo | Base |
|------|------|
| anomalyco/opencode | `dev` |
| huggingface/smolagents | `main` |
| langchain-ai/langgraph | `main` |
| langchain-ai/langchain | `master` |
| pydantic/pydantic-ai | `main` |

### If you accidentally open a PR against levgiorg/<repo>
```bash
gh pr close <N> -R levgiorg/<repo>
gh pr create --repo <UPSTREAM> --head levgiorg:<branch> ...
```

## PR Body: anomalyco/opencode (STRICT — compliance bot enforces exact headers)

```
### Issue for this PR

Closes #<N>

### Type of change

- [x] Bug fix

### What does this PR do?

<1-2 sentences. No walls of text.>

### How did you verify your code works?

- [x] TypeScript typecheck passes
- [x] Existing tests pass

### Screenshots / recordings

_Not applicable — no UI change._

### Checklist

- [x] I have tested my changes locally
- [x] I have not included unrelated changes in this PR
```

## PR Body: Python repos (smolagents, langgraph, langchain)

```
## Summary

<2-3 sentences: what was broken, what you changed, why correct>

## Root Cause

`<file>:<line>` in `<function>()`

## Changes

- `<file>`: <what was changed and why>

## Testing

- [x] All tests pass: `pytest tests/ -x -q`
- [x] Linter clean: `ruff check` / `ruff format`

## Related Issue

Closes #<N>
```

## CI Verification Before Opening

```bash
# TypeScript/Bun repos
export PATH="$HOME/.bun/bin:$PATH"
bun run typecheck && bun test

# Python repos
ruff format . --check && ruff check . && pytest tests/ -x -q
```

## Handling Auto-Close Bots (langgraph/langchain)

1. Comment on issue: "Hi! I'd like to work on this. Could a maintainer assign me?"
2. Open the PR anyway (it will be auto-closed)
3. After close, comment on the ISSUE: "My PR #XXXX was auto-closed. Fix is ready — please assign me or reopen."
4. Update `contribution_log.json` with `"ci_status": "closed"` and `"skip_reason": "auto-closed: contributor not assigned..."`

## Updating PR Body (fix compliance label)

```bash
gh api repos/<owner>/<repo>/pulls/<N> --method PATCH \
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

## Anti-patterns

1. AI-generated walls of text → ignored or closed
2. Missing "Closes #N" → won't auto-close issue on merge
3. Wrong base branch → PR targets wrong base
4. PR against fork → no one sees it
5. Huge diffs from `bun install` → always discard lockfile changes
6. No verification explanation → PR body must say how you tested
