# OSS Contributor

Autonomous workflow for contributing to open source repositories. Never pauses for confirmation.

## Pre-flight

```bash
gh auth status
git config --global user.name "George Levis"
git config --global user.email "levgiorg@gmail.com"
python --version  # 3.10+
export PATH="$HOME/.bun/bin:$PATH"
bun --version     # 1.3+
```

## Workflow

### 1. Fork the repo
```bash
gh repo fork <owner>/<repo> --clone=false
```

### 2. Clone fork, set upstream
```bash
gh repo clone levgiorg/<repo> /tmp/<repo>-<issue#>
cd /tmp/<repo>-<issue#>
git remote add upstream https://github.com/<owner>/<repo>.git
git fetch upstream
```
If `gh repo clone` fails, use: `git clone --depth 1 https://github.com/levgiorg/<repo>.git`

### 3. Create branch from correct base
```bash
# anomalyco/opencode:
git checkout -b fix/issue-<N>-<slug> upstream/dev
# directus/directus, huggingface/smolagents, langchain-ai/langgraph:
git checkout -b fix/issue-<N>-<slug> upstream/main
# langchain-ai/langchain:
git checkout -b fix/issue-<N>-<slug> upstream/master
```

### 4. Install deps — NEVER commit lockfiles

**For opencode (Bun):**
```bash
export PATH="$HOME/.bun/bin:$PATH"
bun install
# IMMEDIATELY discard lockfile changes:
git checkout -- bun.lock 2>/dev/null
git checkout -- packages/*/package.json 2>/dev/null
git clean -fd 2>/dev/null
```

**For directus (pnpm):**
```bash
pnpm install
# Discard lockfile changes:
git checkout -- pnpm-lock.yaml 2>/dev/null
```

**For Python repos (smolagents, langgraph, langchain):**
```bash
pip install -e . 2>/dev/null || pip install -r requirements.txt 2>/dev/null
```

### 5. READ CONTRIBUTING RULES + PR TEMPLATE (CRITICAL — DO NOT SKIP)

```bash
cat CONTRIBUTING.md 2>/dev/null || cat .github/CONTRIBUTING.md 2>/dev/null
cat .github/pull_request_template.md 2>/dev/null || cat .github/PULL_REQUEST_TEMPLATE.md 2>/dev/null
cat AGENTS.md 2>/dev/null
cat .github/workflows/review.yml 2>/dev/null
```

Extract: formatter, linter, type checker, test command, branch naming, base branch, CLA required, auto-close rules. Copy PR template sections into `/tmp/pr_body.md` BEFORE writing any code.

### 6. Make the fix
- Touch only files directly responsible for the bug
- Match existing code style exactly
- Minimal change, no unrelated refactors

### 7. Verify locally
```bash
# TypeScript/Bun:
export PATH="$HOME/.bun/bin:$PATH"
bun run typecheck && bun test
# Python:
ruff format . --check && ruff check . && pytest tests/ -x -q
```

### 7b. For directus/directus — create a changeset
```bash
cat > .changeset/<random-slug>.md << 'EOF'
---
'@directus/app': patch
---

<brief description of the fix>
EOF
```

### 8. Commit
```
fix(<scope>): <description ≤72 chars>

<What was broken, what you changed, why correct. 2-3 sentences.>

Fixes #<N>
```

### 9. Push + PR

```bash
export PATH="$HOME/.bun/bin:$PATH"
git push origin fix/issue-<N>-<slug> --force

gh pr create \
  --repo <upstream-owner>/<upstream-repo> \
  --head levgiorg:<branch-name> \
  --base <dev|main|master> \
  --title "<conventional-commit-title>" \
  --body-file /tmp/pr_body.md
```

**NEVER** omit `--repo <upstream>` — default creates PR against your fork.

### 10. Post-PR: immediately monitor
Run `/ci-guardian` or use `gh pr checks <N> -R <owner>/<repo> --watch --interval 30`.

### 11. Update contribution_log.json

Add an entry:
```json
{
  "repo": "<owner>/<repo>",
  "issue": <N>,
  "issue_title": "<title>",
  "branch": "fix/issue-<N>-<slug>",
  "pr_url": "<url>",
  "pr_number": <N>,
  "ci_status": "pending",
  "opened_at": "<YYYY-MM-DD>",
  "skipped": false,
  "skip_reason": null
}
```

## Anti-patterns

1. **DON'T skip reading CONTRIBUTING.md and PR template** — auto-close in 2h if incomplete
2. **DON'T commit lockfile changes** — huge diffs, CI will flag it
3. **DON'T create PRs against your fork** — always use `--repo <upstream>`
4. **DON'T use SSH** for clones — use HTTPS explicitly
5. **DON'T assume base branch** — always verify from CONTRIBUTING.md
6. **DON'T touch files outside the fix scope** — bots flag suspiciously large PRs
