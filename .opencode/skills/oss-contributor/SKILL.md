# OSS Contributor Skill

Autonomous workflow for contributing to open source repositories. Never pauses for confirmation.

## Pre-flight

```bash
gh auth status
git config --global user.name "George Levis"
git config --global user.email "levgiorg@gmail.com"
python --version  # 3.10+
bun --version     # must be installed
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
**IMPORTANT**: Use HTTPS URLs, not SSH. `gh repo clone` may default to SSH — if it fails, use `git clone --depth 1 https://github.com/levgiorg/<repo>.git`.

### 3. Create branch from correct base
```bash
# For anomalyco/opencode:
git checkout -b fix/issue-<N>-<slug> upstream/dev
# For langchain-ai repos:
git checkout -b fix/issue-<N>-<slug> upstream/main
# For huggingface/smolagents:
git checkout -b fix/issue-<N>-<slug> upstream/main
```

### 4. Install deps (never commit lockfiles!)
```bash
# Python repos — use the repo's tool (uv/pip/make)
# TypeScript/Bun repos:
export PATH="$HOME/.bun/bin:$PATH"
bun install
```

**CRITICAL**: After `bun install`, discard lockfile changes:
```bash
git checkout -- bun.lock package.json 2>/dev/null
git checkout -- packages/*/package.json 2>/dev/null
git clean -fd 2>/dev/null
```

### 5. READ CONTRIBUTING RULES + PR TEMPLATE (CRITICAL — DO NOT SKIP)

**Before writing a single line of code**, read how this repo expects contributions:
```bash
# Read the contributing guide
cat CONTRIBUTING.md 2>/dev/null || cat .github/CONTRIBUTING.md 2>/dev/null

# Read the PR template — MUST match it exactly
cat .github/pull_request_template.md 2>/dev/null || cat .github/PULL_REQUEST_TEMPLATE.md 2>/dev/null

# For sst/opencode: also read AGENTS.md (CI enforces it)
cat AGENTS.md 2>/dev/null

# For sst/opencode: check the review workflow (bot reviews your PR)
cat .github/workflows/review.yml 2>/dev/null
cat .github/workflows/pr-standards.yml 2>/dev/null
```

**Extract from these files:**
- Formatter command (ruff/black/biome/eslint)
- Linter command
- Type checker command (mypy/pyright/tsgo)
- Test command (exact pytest invocation or bun test)
- Branch naming convention
- Commit message format (conventional commits?)
- CLA required? yes/no
- PR template fields (copy them into your pr_body.md BEFORE writing code)
- Auto-close rules (e.g., "must be assigned to issue")
- Base branch (dev/main/master — do not assume)

**For opencode specifically:**
- The PR template `.github/pull_request_template.md` has EXACT required sections
- Missing or incomplete sections trigger the `needs:compliance` label → auto-close in 2 hours
- Template sections: `### Issue for this PR`, `### Type of change`, `### What does this PR do?`, `### How did you verify your code works?`, `### Screenshots / recordings`, `### Checklist`
- `AGENTS.md` style rules are enforced by the review bot

### 6. Make the fix
- Touch only files directly responsible for the bug
- Follow the repo's code style exactly
- Match existing patterns in the same file
- Minimal change, no unrelated refactors

### 7. Verify locally
```bash
# Python: ruff format . && ruff check . && pytest tests/ -x
# TypeScript: bun run typecheck && bun test
```

### 8. Commit
```
fix(<scope>): <description>

<What was broken, what you changed, why correct. 2-3 sentences.>

Fixes #<N>
```
Conventional commit format. Subject ≤72 chars. `Fixes #N` on its own line.

### 9. Push + PR
```bash
export PATH="$HOME/.bun/bin:$PATH"  # needed for husky pre-push hook
git push origin fix/issue-<N>-<slug> --force
```

**CRITICAL**: Always create cross-repo PRs explicitly:
```bash
gh pr create \
  --repo <upstream-owner>/<repo> \
  --head levgiorg:<branch> \
  --base <dev|main> \
  --title "<title>" \
  --body-file /tmp/pr_body.md
```
**NEVER** create PRs against `levgiorg/<repo>` — always cross-reference to upstream.

### 10. MONITOR THE PR IMMEDIATELY (CRITICAL)
```bash
# Watch CI
gh pr checks <pr_number> -R <owner>/<repo> --watch --interval 30

# Check ALL comments (bots + maintainers)
gh pr view <pr_number> -R <owner>/<repo> --json comments \
  --jq '.comments[] | "\(.author.login): \(.body[:300])"'

# Check for compliance labels (opencode: "needs:compliance" = auto-close in 2h)
gh pr view <pr_number> -R <owner>/<repo> --json labels --jq '.labels[].name'
```

**Respond to issues within minutes:**
- `needs:compliance` label → update PR body immediately
- Bot says "duplicate found" → check the flagged PR, close yours if identical
- Maintainer review → fix what they ask, don't argue
- Auto-close (langgraph/langchain) → comment on the ISSUE referencing your closed PR number

### 10. PR Body — Use the ACTUAL template from the repo
For anomalyco/opencode (uses specific template):
```
### Issue for this PR
Closes #<N>

### Type of change
- [x] Bug fix

### What does this PR do?
<clear 1-2 sentence description>

### How did you verify your code works?
- [x] TypeScript typecheck passes
- [x] Existing tests pass

### Checklist
- [x] I have tested my changes locally
- [x] I have not included unrelated changes in this PR
```

For Python repos (smolagents, langchain, langgraph):
```
## Summary
<2-3 sentences: what was broken, what you changed>

## Root Cause
<exact file/function/line responsible>

## Changes
- `<file>`: <what and why>

## Testing
- [x] All tests pass
- [x] Linter clean

## Related Issue
Closes #<N>
```

## Anti-patterns Learned

1. **DON'T skip reading CONTRIBUTING.md and PR template** — opencode auto-closes PRs with incomplete templates. Always `cat .github/pull_request_template.md` FIRST.
2. **DON'T commit lockfile changes** — `bun install` in TypeScript repos generates huge diffs. Always discard them.
3. **DON'T create PRs against your fork** — always use `--repo <upstream>` flag and `--head levgiorg:`.
4. **DON'T use SSH** for clones — use HTTPS URLs explicitly.
5. **DON'T run `bun install` from repo root** if it triggers code generation (like `./script/generate.ts`).
6. **DON'T commit after `bun install` runs husky** — husky needs `bun` in PATH during push.
7. **DON'T touch files outside the fix scope** — CI auto-close bots flag suspiciously large PRs.
8. **DON'T assume base branch** — verify with `cat .github/pull_request_template.md` or check CONTRIBUTING.md. opencode uses dev, not main.
