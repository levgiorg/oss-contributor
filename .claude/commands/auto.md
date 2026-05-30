# Autonomous OSS Contribution Loop

Full-automation bootstrap. Launches the scout → fix → PR → monitor pipeline. Never pauses for confirmation. Runs until target PR count is met or user interrupts.

## How to start

```
/auto
```

Or set it as the default on session start by adding to CLAUDE.md:
```
On startup, run /auto.
```

---

## Phase 0 — Pre-flight (run once)

```bash
gh auth status
git config --global user.name "George Levis"
git config --global user.email "levgiorg@gmail.com"
export PATH="$HOME/.bun/bin:$PATH"
bun --version
python --version
```

If any tool is missing: install it, then continue.

---

## Phase 1 — Assess current state

Read `contribution_log.json` from the workspace root. Count:
- Active PRs (not skipped, ci_status ≠ merged/closed)
- Skipped/closed PRs
- PRs per repo

```bash
node -e "
const fs = require('fs');
const log = JSON.parse(fs.readFileSync('contribution_log.json', 'utf8'));
const active = log.filter(e => !e.skipped && e.ci_status !== 'merged');
const skipped = log.filter(e => e.skipped);
console.log('ACTIVE:' + active.length);
console.log('SKIPPED:' + skipped.length);
console.log('TARGET:15');
console.log('NEED:' + Math.max(0, 15 - active.length));
"
```

**Decision tree:**
- Active PRs < target → go to Phase 2 (scout & fix)
- Active PRs ≥ target → skip to Phase 4 (monitor only)

---

## Phase 2 — Scout for issues

For each repo in the Quick Reference table (CLAUDE.md), run the `/issue-scout` deduplication check:

1. List open bug/issues:
   ```bash
   gh issue list -R <owner>/<repo> -l bug -L 30 --search "no:assignee"
   ```

2. For each candidate, run triple dedup:
   - Check 1: Any open PR referencing this issue? (`gh pr list -R <repo> --search "fixes #<N> OR closes #<N>" --state open`)
   - Check 2: Any cross-referenced PR in timeline? (`gh api "repos/<repo>/issues/<N>/timeline"`)
   - Check 3: Anyone claim it in last 7 days? (check comments for "working on", "PR #", "assigned")

3. Score candidates by ease:
   - Single-file change: +3
   - Clear reproduction: +2
   - Good first issue label: +1
   - TypeScript: +1 (we know this stack)
   - Frontend-only (no API/DB changes): +1

4. Sort by score, pick the top N where N = PRs needed to reach target

5. For each selected issue, invoke `/oss-contribute` or run the full workflow inline:
   - Fork → clone → branch → read templates → fix → test → commit → push → cross-fork PR → log

**Repo priority order** (highest success rate first):
1. **directus/directus** — no auto-close, no assignment needed, CLA already signed
2. **anomalyco/opencode** — strict template but predictable, no auto-close
3. **huggingface/smolagents** — first-time CI needs maintainer approval
4. **langchain-ai/langgraph** — auto-close bot, only if we can get assignment first

---

## Phase 3 — Post-PR actions

After each PR opens:
1. Comment on the parent issue: "Opened PR #<N> to fix this."
2. Append to `contribution_log.json`
3. Run CI checks immediately:
   ```bash
   gh pr checks <N> -R <repo> --watch --interval 30
   ```

If CI fails:
- Check the failure reason
- If fixable autonomously → fix, amend commit, force-push
- If needs external action (maintainer approval, CLA, etc.) → note it and move on

---

## Phase 4 — Monitor loop (persistent)

Start the PR monitor. Use **subagents in parallel** for efficiency — one agent per repo.

### 4a — Gather state (parallel subagents)

Spin up one agent per repo to check:
- PR state (open/closed/merged)
- New comments from bots or humans
- Labels
- CI status
- Maintainer reviews

### 4b — Act on findings

| Finding | Auto-fix? | Action |
|---------|-----------|--------|
| `needs:compliance` label | ✅ YES | Patch PR body with correct template sections via `gh api PATCH` |
| Auto-close by bot | ✅ YES | Comment on parent issue requesting assignment, mark skipped |
| Potential duplicate flag | ✅ YES | Check flagged PR; if identical → close ours; if different → comment why |
| CI failure from our code | ✅ YES | Analyze failure, fix, amend, force-push |
| Simple human comment ("missing X", "add Y") | ✅ YES | Clone branch, make the change, push |
| Complex human review ("redesign X", "rethink approach") | ❌ NO | Surface to user with clear summary |
| PR merged | ✅ YES | Update log: `ci_status = "merged"`, celebrate |
| All green, no activity | — | Wait, do not ping maintainers |

### 4c — Auto-respond to human review comments

When a reviewer says something like "This PR seems to be missing X":

1. Read the comment carefully — is the request clear and scoped?
2. If YES: clone the branch, make the fix, commit, push
3. Reply to the comment: "Good catch — added in [commit hash]. Thanks!"
4. If UNCLEAR: ask for clarification, do NOT guess

### 4d — Repair broken PRs

Some PRs break due to template issues (like #29406). When a PR is auto-closed with `needs:compliance` or `needs:issue`:

1. Patch the PR body with the correct template
2. Link the issue correctly: add `Closes #<N>` or `Fixes #<N>`
3. The PR may auto-reopen, or we open a fresh one pointing at the same branch

---

## Phase 5 — Loop

After completing all phases:

```bash
node -e "
const fs = require('fs');
const log = JSON.parse(fs.readFileSync('contribution_log.json', 'utf8'));
const active = log.filter(e => !e.skipped && e.ci_status !== 'merged');
console.log('Active: ' + active.length + ' / Target: 15');
if (active.length < 15) console.log('LOOP: need ' + (15 - active.length) + ' more');
else console.log('DONE: target reached');
"
```

- If still below target → go back to Phase 2 (scout more issues)
- If at target → persist the monitor and idle until PRs need attention

### Self-pacing

Use `ScheduleWakeup` to continue the loop. The `/loop dynamic` mode is ideal:
```
/loop /auto
```

This re-enters `/auto` each cycle, checks state, and acts accordingly.

---

## Anti-patterns

1. **DON'T wait for user confirmation** — the entire point is 0-touch
2. **DON'T stop at the first failure** — one bad issue shouldn't block the pipeline
3. **DON'T ping maintainers** — "any update?" comments hurt our chances
4. **DON'T fix issues assigned to maintainers** — they're claimed
5. **DON'T submit feature PRs** — bugs and small improvements only
6. **DON'T touch more than 5 files per fix** — CI bots flag large PRs
7. **DON'T guess on unclear review feedback** — ask for clarification
8. **DON'T skip the triple dedup check** — duplicate PRs waste everyone's time
