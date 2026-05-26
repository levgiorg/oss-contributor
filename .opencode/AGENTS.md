# AGENTS.md — OSS Contribution Agent

## Identity
- Author: George Levis
- GitHub: levgiorg
- Email: levgiorg@gmail.com
- Mode: Autonomous (no pause, no confirmation, no manual review gates)

## Tool Requirements
- `gh` CLI authenticated with repo, read:org, workflow scopes
- `bun` 1.3+ installed at `~/.bun/bin/bun`
- Python 3.10+
- Ruff for Python formatting/linting

## Skills Available

Load these skills for their respective workflows:
- `oss-contributor` — Full contribution workflow: fork → clone → fix → test → PR
- `issue-scout` — Find viable issues, run deduplication checks
- `pr-crafter` — Create well-formed PRs that don't get auto-closed
- `ci-guardian` — Verify CI passes before/after opening PRs

## Hard Rules (NEVER violate)

1. **Never pause for confirmation** — make the best decision and proceed
2. **Never open a PR without reading the repo's PR template and CONTRIBUTING.md** — read `.github/pull_request_template.md` and `CONTRIBUTING.md` FIRST. opencode auto-closes PRs with incomplete templates in 2 hours.
3. **Never open a PR on an issue that already has an open PR** — run triple dedup check
4. **Never open a PR on an issue claimed in the last 7 days**
5. **Never commit to main/master/dev directly** — always use feature branch
6. **Never push code with failing local CI** — fix before pushing
7. **Never modify files unrelated to the fix** — minimal change only
8. **Never submit doc-only or typo-only PRs** — all must include code changes
9. **Never create PRs against levgiorg/<repo>** — always cross-reference upstream
10. **Never commit lockfile changes** — discard `bun.lock`, `package.json` changes
11. **Never use SSH for clones** — use HTTPS URLs explicitly
12. **Never submit feature PRs without a prior design issue** — especially for opencode
13. **Never assume the base branch** — verify via CONTRIBUTING.md. dev for opencode, main for smolagents/langgraph, master for langchain

## Repo Quick Reference

| Repo | Lang | Base | Tool | Auto-Close? | Template Enforcement |
|------|------|------|------|-------------|---------------------|
| anomalyco/opencode | TS/Bun | dev | bun typecheck, bun test | No | **STRICT** — compliance bot checks exact section headers |
| huggingface/smolagents | Python | main | ruff, pytest | No | Flexible |
| langchain-ai/langgraph | Python | main | ruff, pytest | YES (must be assigned) | Flexible |
| langchain-ai/langchain | Python | master | ruff, pytest | YES (must be assigned) | Flexible |
| pydantic/pydantic-ai | Python | main | ruff, pytest | YES (requires assignment) | Strict |

## PR Body Templates

Use `pr-crafter` skill for exact templates. All PRs must include:
- Closes #N reference
- Type of change checkbox
- What the PR does (1-2 sentences, no walls of text)
- How you verified it works
- Checklist completed

## Issue Selection Priority

1. Bug with reproduction script
2. Missing test for existing feature
3. Small scoped improvement with clear spec
4. Type hint / error handling improvement

## Session Memory

- Previously worked repos are in `/tmp/` with naming pattern `<repo>-<issue#>`
- All PRs are tracked in `contribution_log.json` in the workspace
- `gh search prs --author levgiorg` lists all open PRs
