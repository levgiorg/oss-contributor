# CLAUDE.md — OSS Contribution Workspace

## Identity
- Author: George Levis
- GitHub: levgiorg
- Email: levgiorg@gmail.com
- Mode: Autonomous (no pause, no confirmation, no manual review gates)

## Tool Requirements

Verify these are available before starting any workflow:

```bash
gh auth status                         # must be authenticated with repo, read:org, workflow scopes
git config user.name                   # must show "George Levis"
export PATH="$HOME/.bun/bin:$PATH"
bun --version                          # 1.3+
python --version                       # 3.10+
ruff --version                         # any recent
```

## Available Commands

| Command | Purpose |
|---------|---------|
| `/oss-contribute` | Full contribution workflow: fork → clone → fix → test → PR |
| `/issue-scout` | Find viable issues, run triple deduplication check |
| `/pr-crafter` | Create well-formed PRs that pass compliance bots |
| `/ci-guardian` | Verify CI passes, fix common failures, monitor post-PR |
| `/monitor-prs` | Check all open PRs for comments, labels, reviews — act on issues |

### Starting the PR Monitor

**Session-bound (while Claude Code is open):**
```
/loop 60m /monitor-prs
```

**Unattended (runs even when session is closed):**
```
/schedule
```
Then describe: "Every 60 minutes, run /monitor-prs against the contribution_log.json in /Users/georgelevis/Projects/Contibuting"

## Hard Rules (NEVER violate)

1. **Never pause for confirmation** — make the best decision and proceed
2. **Never open a PR without reading the repo's PR template and CONTRIBUTING.md** — read `.github/pull_request_template.md` and `CONTRIBUTING.md` FIRST
3. **Never open a PR on an issue that already has an open PR** — run triple dedup check
4. **Never open a PR on an issue claimed in the last 7 days**
5. **Never commit to main/master/dev directly** — always use feature branch
6. **Never push code with failing local CI** — fix before pushing
7. **Never modify files unrelated to the fix** — minimal change only
8. **Never submit doc-only or typo-only PRs** — all must include code changes
9. **Never create PRs against levgiorg/<repo>** — always cross-reference upstream
10. **Never commit lockfile changes** — discard `bun.lock`, `package.json` changes
11. **Never use SSH for clones** — use HTTPS URLs explicitly
12. **Never submit feature PRs without a prior design issue**
13. **Never assume the base branch** — verify via CONTRIBUTING.md. `dev` for opencode, `main` for smolagents/langgraph, `master` for langchain

## Repo Quick Reference

| Repo | Lang | Base | CI Tool | Auto-Close? | Template |
|------|------|------|---------|-------------|---------|
| anomalyco/opencode | TS/Bun | `dev` | `bun typecheck`, `bun test` | No | **STRICT** — compliance bot checks exact section headers |
| huggingface/smolagents | Python | `main` | `ruff`, `pytest` | No | Flexible |
| langchain-ai/langgraph | Python | `main` | `ruff`, `pytest` | YES (must be assigned) | Flexible |
| langchain-ai/langchain | Python | `master` | `ruff`, `pytest` | YES (must be assigned) | Flexible |
| pydantic/pydantic-ai | Python | `main` | `ruff`, `pytest` | YES (requires assignment) | Strict |

## PR Body Templates

### anomalyco/opencode (STRICT — compliance bot enforces exact headers)

```
### Issue for this PR

Closes #<N>

### Type of change

- [x] Bug fix

### What does this PR do?

<1-2 sentences describing the fix. No walls of text.>

### How did you verify your code works?

- [x] TypeScript typecheck passes
- [x] Existing tests pass

### Screenshots / recordings

_Not applicable — no UI change._

### Checklist

- [x] I have tested my changes locally
- [x] I have not included unrelated changes in this PR
```

### Python repos (smolagents, langgraph, langchain)

```
## Summary

<2-3 sentences: what was broken, what you changed, why correct>

## Root Cause

`<file>:<line>` in `<function>()`

## Changes

- `<file>`: <what and why>

## Testing

- [x] All tests pass: `pytest tests/ -x -q`
- [x] Linter clean: `ruff check` / `ruff format`

## Related Issue

Closes #<N>
```

## Issue Selection Priority

1. Bug with reproduction script
2. Missing test for existing feature
3. Small scoped improvement with clear spec
4. Type hint / error handling improvement

## Session Memory

- Cloned repos live in `/tmp/` with naming pattern `<repo>-<issue#>`
- All PRs are tracked in `contribution_log.json` in this workspace root
- `gh search prs --author levgiorg` lists all open PRs
- `/monitor-prs` reads `contribution_log.json` and acts on live PR state
