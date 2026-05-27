# OSS Contribution Workspace — From Scratch

A step-by-step guide for anyone cloning this repo and using it to contribute to open source projects autonomously via Claude Code.

---

## Prerequisites

Install these before anything else:

```bash
# 1. GitHub CLI
brew install gh                         # macOS
# or: https://cli.github.com/

# 2. Authenticate gh (needs repo, read:org, workflow scopes)
gh auth login
gh auth status                          # verify

# 3. Bun (TypeScript repos use it)
curl -fsSL https://bun.sh/install | bash
export PATH="$HOME/.bun/bin:$PATH"     # add to ~/.zshrc or ~/.bashrc
bun --version                           # verify 1.3+

# 4. Python 3.10+
python3 --version                       # verify

# 5. Ruff (Python linter/formatter)
pip install ruff
ruff --version                          # verify

# 6. Claude Code CLI
npm install -g @anthropic-ai/claude-code
```

---

## Step 1 — Clone and open the workspace

```bash
git clone https://github.com/levgiorg/oss-contributor.git
cd oss-contributor
claude .
```

Claude Code will automatically load:
- `.claude/CLAUDE.md` — hard rules, repo table, PR templates
- `.claude/commands/` — all slash commands
- `.claude/settings.json` — pre-approved permissions (no prompts for `gh`, `git`, `bun`, `ruff`, `pytest`)

---

## Step 2 — Personalise for your GitHub account

Open `.claude/CLAUDE.md` and update the **Identity** section:

```markdown
## Identity
- Author: Your Name
- GitHub: yourgithubhandle          ← change this
- Email: you@email.com              ← change this
- Mode: Autonomous
```

Also update every occurrence of `levgiorg` in the command files to your GitHub handle. The critical places are:

| File | What to change |
|------|---------------|
| `.claude/commands/oss-contribute.md` | `gh repo clone levgiorg/<repo>` → your handle |
| `.claude/commands/oss-contribute.md` | `--head levgiorg:<branch>` → your handle |
| `.claude/commands/pr-crafter.md` | `--head levgiorg:<branch>` → your handle |
| `.claude/commands/pr-crafter.md` | `gh pr close <N> -R levgiorg/<repo>` → your handle |
| `.claude/commands/issue-scout.md` | `gh search prs --author levgiorg` → your handle |
| `.claude/commands/ci-guardian.md` | `gh search prs --author levgiorg` → your handle |

> **Tip**: Run a global find-and-replace in your editor: `levgiorg` → `yourgithubhandle`

---

## Step 3 — Add the repos you want to contribute to

Open `.claude/CLAUDE.md` and edit the **Repo Quick Reference** table.

Add a row for each repo:

```markdown
| owner/repo | Lang | base-branch | CI command | Auto-Close? | Template enforcement |
```

**Example:**

```markdown
| anomalyco/opencode     | TS/Bun | `dev`    | `bun typecheck`, `bun test` | No  | STRICT |
| huggingface/smolagents | Python | `main`   | `ruff`, `pytest`            | No  | Flexible |
| langchain-ai/langgraph | Python | `main`   | `ruff`, `pytest`            | YES | Flexible |
| langchain-ai/langchain | Python | `master` | `ruff`, `pytest`            | YES | Flexible |
| pydantic/pydantic-ai   | Python | `main`   | `ruff`, `pytest`            | YES | Strict |
```

**Fields explained:**

| Field | What to fill |
|-------|-------------|
| `owner/repo` | Exact GitHub path, e.g. `facebook/react` |
| `Lang` | `Python`, `TS/Bun`, `Go`, etc. |
| `base-branch` | The branch PRs target — check `CONTRIBUTING.md` in the repo |
| `CI command` | How to run tests locally — check `CONTRIBUTING.md` |
| `Auto-Close?` | Does the repo bot auto-close unassigned PRs? Check issues for "bot" comments. |
| `Template` | `STRICT` if the repo has a compliance bot; `Flexible` otherwise |

> **How to find the base branch**: open the repo on GitHub → Pull Requests → any merged PR → look at `base: dev` or `base: main` in the PR header.

---

## Step 4 — Find issues to work on

```
/issue-scout
```

Tell Claude which repo:
> "Scout `anomalyco/opencode` for bugs. Skip anything claimed in the last 7 days."

Or search multiple repos:
> "Scout all repos in our table. Give me the top 3 viable issues sorted by ease."

Claude will run the **triple dedup check** for each candidate:
1. No existing open PR for that issue
2. No cross-referenced PR in the issue timeline
3. No ownership claim comment in the last 7 days

It returns a ranked list. Pick one.

---

## Step 5 — Open a PR

```
/oss-contribute
```

Tell Claude the target:
> "Contribute to `anomalyco/opencode` issue #29500"

Claude will autonomously:
1. Fork the repo under your handle (if not already forked)
2. Clone to `/tmp/<repo>-<issue#>`
3. Read `CONTRIBUTING.md` and the PR template
4. Create a feature branch from the correct base
5. Make the minimal fix
6. Run local CI (typecheck + tests / ruff + pytest)
7. Commit with conventional commit format
8. Push and open a cross-fork PR against upstream
9. Append an entry to `contribution_log.json`

**If you want to target a specific issue manually**, just specify it:
> "Fix issue #29500 in `anomalyco/opencode` — the bug is about viewport snapping during history reads"

---

## Step 6 — Start monitoring your PRs

### Option A — Session-bound (while Claude Code is open)

```
/loop 60m /monitor-prs
```

Every 60 minutes Claude will:
- Check every PR in `contribution_log.json` for new comments, labels, and reviews
- **Auto-fix** `needs:compliance` labels by patching the PR body
- **Auto-comment** on parent issues when a bot auto-closes a PR
- **Surface** (but not auto-fix) maintainer code review requests
- Print a one-line status table

### Option B — Unattended (runs even when your terminal is closed)

```
/schedule
```

When prompted, describe the routine:
> "Every 60 minutes, run /monitor-prs in /Users/yourname/Projects/oss-contributor"

You'll get a notification when action is needed.

---

## Understanding the status table

Each loop cycle prints:

```
=== PR Monitor — 2026-05-28 14:00 ===
anomalyco/opencode#29500  — FIXED    — needs:compliance label cleared
smolagents#2314           — PENDING  — CI awaiting maintainer approval
langgraph#7920            — CLOSED   — commented on issue #7800 requesting assignment
opencode#29501            — REVIEW   — maintainer requested changes (see PR)
```

| Status | Meaning | Action |
|--------|---------|--------|
| `FIXED` | Claude auto-fixed a compliance or bot issue | Nothing — done |
| `PENDING` | Waiting on CI or maintainer | Wait |
| `CLOSED` | Bot auto-closed (auto-close repo) | Wait for maintainer assignment |
| `REVIEW` | Maintainer left code feedback | Push a fix commit to the branch |
| `MERGED` | PR was merged | Nothing — log updated automatically |

---

## The contribution log

`contribution_log.json` is the source of truth for all open PRs. The monitor reads it on every cycle.

**Format:**
```json
[
  {
    "repo": "anomalyco/opencode",
    "issue": 29500,
    "issue_title": "short description",
    "branch": "fix/issue-29500-slug",
    "pr_url": "https://github.com/anomalyco/opencode/pull/29510",
    "pr_number": 29510,
    "ci_status": "pending",
    "opened_at": "2026-05-28",
    "skipped": false,
    "skip_reason": null
  }
]
```

**To stop monitoring a PR** (e.g. you decided to close it): set `"skipped": true`.

**`ci_status` values** that Claude updates automatically:

| Value | Meaning |
|-------|---------|
| `pending` | Open, CI running or waiting |
| `closed` | Auto-closed by bot (commented on issue) |
| `merged` | Merged by maintainer |

---

## Quick command reference

| Command | What to say after invoking |
|---------|---------------------------|
| `/issue-scout` | "Scout `owner/repo` for bugs" |
| `/oss-contribute` | "Contribute to `owner/repo` issue #N" |
| `/pr-crafter` | "Write the PR body for issue #N in `owner/repo`" |
| `/ci-guardian` | "Check CI for PR #N in `owner/repo`" |
| `/monitor-prs` | (no args needed — reads contribution_log.json) |
| `/loop 60m /monitor-prs` | (starts the monitoring loop) |
| `/schedule` | Describe the cron routine in plain language |

---

## Repos to avoid (structural barriers)

| Repo | Why |
|------|-----|
| `langchain-ai/langchain` | Auto-close bot + CLA + 5+ competitors per issue |
| `pydantic/pydantic-ai` | Requires explicit maintainer pre-assignment — not automatable |

---

## Common mistakes

| Mistake | What happens | Fix |
|---------|-------------|-----|
| Forgot to read PR template | Compliance bot adds `needs:compliance` label → auto-close in 2h | `/monitor-prs` fixes it automatically |
| PR opened against your fork | No one sees it | `/pr-crafter` — always uses `--repo upstream` flag |
| Committed `bun.lock` | 300+ file diff, CI flags it | Force-push a clean branch |
| Wrong base branch | PR targets wrong branch | Check `CONTRIBUTING.md` first |
| SSH clone fails | `gh repo clone` defaults to SSH | Use `git clone https://...` explicitly |
| Bun not in PATH during push | Husky pre-push hook fails | `export PATH="$HOME/.bun/bin:$PATH"` before push |
