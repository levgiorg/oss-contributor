# Contribution Mission — Handoff Report

**Date**: 2026-05-26
**Author**: George Levis (levgiorg)

---

## Summary

| Repo | Open PRs | Closed PRs | Issues |
|------|----------|------------|--------|
| anomalyco/opencode | 12 | 3 (fork cleanup) | 12 |
| huggingface/smolagents | 3 | 0 | 3 |
| langchain-ai/langgraph | 0 | 3 (auto-close bot) | 3 |
| langchain-ai/langchain | 0 | 0 | Blocked |
| pydantic/pydantic-ai | 0 | 0 | Blocked |
| **TOTAL** | **15** | **3** | **18 attempts** |

---

## ALL OPEN PRs

### anomalyco/opencode (12 PRs — BASE: dev)

| # | Issue | Title | URL |
|---|-------|-------|-----|
| 29370 | #29277 | /compact works when default_agent is subagent | https://github.com/anomalyco/opencode/pull/29370 |
| 29371 | #29290 | case-insensitive pwsh env var lookup | https://github.com/anomalyco/opencode/pull/29371 |
| 29372 | #29291 | byte length for preview truncation | https://github.com/anomalyco/opencode/pull/29372 |
| 29378 | #29330 | drain stdout in export to prevent truncation | https://github.com/anomalyco/opencode/pull/29378 |
| 29379 | #29350 | handle missing subagent session in Task messages | https://github.com/anomalyco/opencode/pull/29379 |
| 29400 | #29390 | output error message on .fail() | https://github.com/anomalyco/opencode/pull/29400 |
| 29401 | #29148 | handle prompt_skills keybind | https://github.com/anomalyco/opencode/pull/29401 |
| 29402 | #29311 | add rtk to bash arity dictionary | https://github.com/anomalyco/opencode/pull/29402 |
| 29403 | #29214 | stabilize flaky shell test | https://github.com/anomalyco/opencode/pull/29403 |
| 29404 | #29366 | handle JSON parse failure in models-dev | https://github.com/anomalyco/opencode/pull/29404 |
| 29406 | #29094 | don't snap viewport while reading history | https://github.com/anomalyco/opencode/pull/29406 |
| 29407 | #29375 | make identity cache channel-specific | https://github.com/anomalyco/opencode/pull/29407 |

**Known issues:**
- #29370 flagged as duplicate of #29276 (identical diff)
- #29372 flagged as potential duplicate of #29297
- #29378 compliance label fixed, now green
- All CI checks passing on all 12 PRs (check-duplicates, check-standards, check-compliance, add-contributor-label)

### huggingface/smolagents (3 PRs — BASE: main)

| # | Issue | Title | URL |
|---|-------|-------|-----|
| 2311 | #1849 | don't pass parent kwargs to managed agents | https://github.com/huggingface/smolagents/pull/2311 |
| 2312 | #2050 | Docker executor cleanup on exit | https://github.com/huggingface/smolagents/pull/2312 |
| 2313 | #2090 | evaluate_with() calls __exit__ on original context manager | https://github.com/huggingface/smolagents/pull/2313 |

**Known issues:**
- CI has not run (first-time contributor — needs maintainer approval)
- All fixes verified locally with ruff + pytest

---

## CLOSED PRs (Auto-Close Bot)

### langchain-ai/langgraph (3 PRs — CLOSED)

| # | Issue | Title | Status |
|---|-------|-------|--------|
| 7909 | #7684 | PostgresStore numeric filter operators | Auto-closed. Commented on #7684 requesting assignment. |
| 7911 | #7686 | broken docs URL in RuntimeError | Auto-closed. Commented on #7686 referencing closed PR. |
| 7912 | #7776 | stacklevel=2 for warnings.warn() | Auto-closed. Commented on #7776 referencing closed PR. |

**Recovery path**: A maintainer must assign @levgiorg to the issue, then the PRs auto-reopen. All 3 issues have comments requesting assignment with PR numbers referenced.

---

## BLOCKED Repos

### langchain-ai/langchain
- Auto-close bot + CLA requirement
- Every viable issue has 3-6 people competing for assignment
- No PRs attempted — structural barrier

### pydantic/pydantic-ai
- Explicit requirement: "Wait for assignment from maintainer before opening a PR"
- "Unassigned PRs may be auto-closed"
- Cannot proceed autonomously — requires human maintainer interaction

---

## WHAT WE LEARNED (Critical Anti-Patterns)

1. **Dirty PRs from bun install**: `bun install` generates 300+ file diffs (lockfiles, SDK regeneration). Always discard non-source changes before committing. Force-pushed fix for #29371.

2. **Wrong PR target**: 3 PRs accidentally opened against `levgiorg/opencode` instead of `anomalyco/opencode`. Must always use `--repo anomalyco/opencode --head levgiorg:...`.

3. **Husky blocks git push**: The opencode repo has a husky pre-push hook needing `bun` in PATH. Always `export PATH="$HOME/.bun/bin:$PATH"` before pushing.

4. **OpenAI compliance bot**: The template check is strict. Missing any section triggers the `needs:compliance` label. Fixed #29378 by updating PR body via API.

5. **Duplicate detection**: OpenCode's own CI detects duplicate PRs. #29370 is an exact duplicate of #29276 — the maintainer will choose one.

6. **Auto-close bots are unbypassable**: LangChain/langgraph bot requires maintainer assignment. The only autonomous workaround is commenting on the issue and accepting the close.

---

## TO CONTINUE THIS MISSION

1. **Load the skills**: `skill oss-contributor`, `skill issue-scout`, `skill pr-crafter`, `skill ci-guardian`
2. **Read AGENTS.md**: Contains repo references, base branches, hard rules
3. **Before working on any issue**: run the triple dedup check (open PRs, cross-references, ownership claims)
4. **For opencode**: Find 3-5 more bugs NOT already covered by the 12 PRs above
5. **For langgraph**: Check if maintainers responded to assignment requests on #7684, #7686, #7776. If assigned, reopen PRs #7909, #7911, #7912.
6. **For langchain/pydantic-ai**: Only viable with maintainer pre-approval — not suitable for autonomous workflow

## Files Created

| File | Path | Purpose |
|------|------|---------|
| AGENTS.md | `.opencode/AGENTS.md` | Autonomous agent rules + repo quick reference |
| oss-contributor skill | `.opencode/skills/oss-contributor/SKILL.md` | Full contribution workflow |
| issue-scout skill | `.opencode/skills/issue-scout/SKILL.md` | Issue discovery + dedup |
| pr-crafter skill | `.opencode/skills/pr-crafter/SKILL.md` | PR creation + templates |
| ci-guardian skill | `.opencode/skills/ci-guardian/SKILL.md` | CI verification + fix patterns |
| contribution_log.json | `contribution_log.json` | Machine-readable PR log |
| HANCOFF.md | `HANCOFF.md` | This file |
