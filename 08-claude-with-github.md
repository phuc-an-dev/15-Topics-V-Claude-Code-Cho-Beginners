# Topic 8: Best Practices Using Claude Với GitHub

> **Nguồn**: Anthropic official docs (code-review plugin, CLAUDE.md), `ykdojo/claude-code-tips` Tip 4, 26, 29

---

## 🎯 Core Rule: **Dùng `gh` CLI, KHÔNG dùng web fetch**

> "Use `gh` CLI to interact with GitHub. **Do not use web fetch**." — Anthropic official code-review plugin

### Vì sao?
- `gh` CLI đã authenticated, không hit rate limits
- Tốn **ít tokens** hơn nhiều so với parse HTML từ web
- Truy cập được cả GraphQL API

### Install
```bash
brew install gh          # macOS
gh auth login            # Authenticate once
```

---

## 🚀 Basic GitHub Workflow Với Claude

### 1. Để Claude handle Git/GitHub commands
```
> Commit các changes với message mô tả
> Push lên feature branch
> Tạo draft PR, request review
```

Claude sẽ:
- `git add -A`
- `git commit -m "..."` (tự viết message)
- `git push -u origin <branch>`
- `gh pr create --draft ...`

### 2. Safety Rules (quan trọng)

| Action | Auto-allow? | Lý do |
|--------|------------|-------|
| `git pull` | ✅ OK | Không phá origin |
| `git push` | ⚠️ Hỏi | Có thể break origin |
| `git reset --hard` | ❌ Hỏi | Mất work |
| `gh pr create --draft` | ✅ OK | Low risk |
| `gh pr merge` | ❌ Hỏi | Thay đổi main branch |
| `git commit` | ✅ OK | Local only |

### 3. Settings.json recommendations
```json
{
  "permissions": {
    "allow": [
      "Bash(git pull:*)",
      "Bash(git status:*)",
      "Bash(git log:*)",
      "Bash(git diff:*)",
      "Bash(gh pr list:*)",
      "Bash(gh issue view:*)"
    ]
  },
  "attribution": {
    "commit": "",
    "pr": ""
  }
}
```

---

## 💡 Common Workflows

### A. Commit với Descriptive Messages

**Prompt**:
```
Commit these changes. Write a descriptive commit message following
conventional commits format.
```

Claude sẽ phân tích diff → viết message kiểu:
```
feat(auth): add Google OAuth with PKCE flow

- Implement OAuth2 authorization code with PKCE
- Add callback handler for /auth/google/callback
- Store tokens securely using HttpOnly cookies
- Add integration tests for happy path + error cases
```

### B. Draft PR → Review → Merge

**Pattern ưa thích**:
```
1. Create draft PR first (low risk)
2. Review changes carefully
3. Run verifications (tests, lint)
4. Convert to ready-for-review
5. Merge
```

**Prompt**:
```
Create a draft PR with:
- Title: feat: add user dashboard
- Description: checklist of what was done
- Link to related issue
Mark as draft so we can review before requesting review.
```

### C. Fix GitHub Issue từ ID

**Skill template `/fix-issue <N>`**:
```
1. gh issue view $ARGUMENTS
2. Understand problem
3. Search codebase for relevant files
4. Implement fix
5. Write tests
6. Verify: lint + typecheck passes
7. Commit with descriptive message
8. Push + create PR linked to issue
```

### D. PR Review (Interactive)

**Prompt**:
```
Review PR #123 in this repo.
First, show me the diff with `gh pr diff 123`.
Then go through file-by-file:
- Point out potential bugs
- Check for security issues
- Verify tests cover changes
- Suggest improvements
```

---

## 🔥 Advanced: GraphQL Queries qua `gh`

### Case: Xem edit history of PR description
```bash
gh api graphql -f query='
  query {
    repository(owner: "owner", name: "repo") {
      pullRequest(number: 123) {
        userContentEdits(first: 100) {
          nodes { editedAt editor { login } }
        }
      }
    }
  }'
```

Claude có thể construct GraphQL queries cho needs đặc biệt.

---

## 🎨 Claude Code Review Plugin (Official)

### Official plugin từ Anthropic
**Repo**: `anthropics/claude-code` - path: `plugins/code-review/`

### Features
- **4 review agents** chạy song song: bugs, security, logic, CLAUDE.md compliance
- **Confidence scoring 0-100** - chỉ report issues ≥80
- **Validation step**: Subagents re-check each issue để lọc false positives
- Uses **Opus** cho bugs/logic, **Sonnet** cho CLAUDE.md checks

### Install
```bash
/plugin marketplace add anthropics/claude-code
/plugin install code-review@anthropics
```

### Usage
```bash
# Review locally (output to terminal)
/code-review

# Post as PR comment
/code-review --comment
```

### Example output
```
## Code review
Found 3 issues:

1. Missing error handling for OAuth callback
   (CLAUDE.md says "Always handle OAuth errors")
   https://github.com/owner/repo/blob/abc123/src/auth.ts#L67-L72

2. Memory leak: OAuth state not cleaned up
   https://github.com/owner/repo/blob/abc123/src/auth.ts#L88-L95

3. Inconsistent naming pattern
   https://github.com/owner/repo/blob/abc123/src/utils.ts#L23-L28
```

### Tune Threshold
Default confidence = 80. Edit `commands/code-review.md`:
```
Filter out any issues with a score less than 80.
```
Change `80` to your preference (0-100).

---

## 🤖 Managed Code Review (Team/Enterprise)

### Anthropic's hosted Code Review
- Auto runs on PR open/update
- Posts inline comments as Claude GitHub bot
- Each finding has severity (Important / Nit / Pre-existing)
- 👍/👎 buttons for feedback (Anthropic uses để tune reviewer)
- Check run appears alongside CI checks (neutral conclusion, never blocks)

### Trigger re-review
Comment `@claude review` như top-level PR comment.

### Machine-readable output
```bash
gh api repos/OWNER/REPO/check-runs/CHECK_RUN_ID \
  --jq '.output.text | split("bughunter-severity: ")[1] | split(" -->")[0] | fromjson'
# Returns: {"normal": 2, "nit": 1, "pre_existing": 0}
```

Use cho gating merges: nếu `normal > 0`, block merge trong CI.

---

## 🛠️ Custom GitHub Workflows via Skills

### Skill: `/dx:gha <url>` (từ ykdojo/claude-code-tips)
**Purpose**: Analyze GitHub Actions failures

```
/dx:gha https://github.com/owner/repo/actions/runs/123456

# Claude will:
# - Fetch run details với gh
# - Check for flakiness (compare with recent runs)
# - Identify breaking commits
# - Suggest fixes
```

### Skill: Auto-commit với conventional commits
Skill file `.claude/skills/conventional-commit/SKILL.md`:
```markdown
---
name: conventional-commit
description: Commit using conventional commits format
---
When asked to commit:
1. Run `git diff --cached` to see staged changes
2. Determine type: feat|fix|docs|style|refactor|test|chore
3. Determine scope từ file paths
4. Write subject (imperative, <50 chars)
5. Add body explaining WHY (not WHAT)
6. Reference issue nếu applicable
7. Commit với format: `<type>(<scope>): <subject>`
```

---

## 📋 CI/CD Integration

### GitHub Actions với Claude
Anthropic provides official GitHub Actions integration:
- Run Claude Code trong CI pipelines
- Non-interactive mode: `claude -p "prompt"`
- Structured output: `--output-format json` hoặc `stream-json`

**Example workflow** `.github/workflows/claude-review.yml`:
```yaml
name: Claude Review
on: [pull_request]
jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: anthropics/claude-code-action@v1
        with:
          prompt: "Review the changes in this PR"
          output-format: "json"
```

### Pre-commit Hooks
```bash
# .git/hooks/pre-commit
#!/bin/bash
claude -p "Review staged changes for obvious issues" --output-format text
```

---

## 🚨 Anti-Patterns

### 1. **Web fetch thay vì `gh`**
❌ "Go to github.com/owner/repo/pull/123 and..."
✅ "Use `gh pr view 123` to..."

### 2. **Auto-approve `gh pr merge`**
❌ Để Claude merge tự động
✅ Draft PR → human review → human merge

### 3. **Commit trực tiếp vào main**
❌ Let Claude push to main branch
✅ Always PR workflow, protected branches

### 4. **Ignore Attribution Settings**
Mặc định Claude thêm "Co-Authored-By: Claude" vào commits.
```json
{ "attribution": { "commit": "", "pr": "" } }
```
→ Remove nếu không muốn attribution.

### 5. **Paste GitHub URLs vào context**
URLs tốn tokens parse.
✅ Dùng `gh issue view` / `gh pr view` để Claude read directly.

### 6. **Không dùng Draft PRs**
Draft PRs = low-risk workflow cho Claude.
- Claude tạo draft → bạn review → mark ready.
- Nếu sai → close draft, không ai thấy.

---

## 🎯 Prompt Templates

### Template 1: Create Feature Branch + PR
```
I'm implementing feature X.
1. Create a new branch `feature/X`
2. Make changes (see my requirements below)
3. Commit with conventional commits
4. Push as draft PR
5. Link to issue #<N>
6. Add checklist to PR description:
   - [ ] Tests added
   - [ ] Documentation updated
   - [ ] No breaking changes
```

### Template 2: Investigate CI Failure
```
CI failing on PR #<N>.
1. Use `gh pr checks <N>` to see failing jobs
2. Use `gh run view <run-id> --log` to see logs
3. Identify root cause
4. Is it flaky or real? Check last 10 runs.
5. If real: fix + push
6. If flaky: document in docs/flaky-tests.md
```

### Template 3: Review Own PR Before Requesting Review
```
Review my open PR #<N> as if you're a strict reviewer:
- Bugs or logic errors?
- Security issues?
- Missing tests?
- Inconsistent with CLAUDE.md rules?
- Performance concerns?
Don't be nice, be thorough.
```

### Template 4: Debug Old Code
```
This bug was introduced sometime recently.
Use `git log --oneline -20` to see recent commits.
Suspect commits that touched <file>.
Use `git bisect` to find exact commit.
Once found, analyze what changed and propose fix.
```

---

## 📊 Useful `gh` Commands Claude Can Use

| Command | Purpose |
|---------|---------|
| `gh pr list` | List open PRs |
| `gh pr view <N>` | View PR details |
| `gh pr diff <N>` | View PR changes |
| `gh pr checks <N>` | CI status |
| `gh pr review <N>` | Add review |
| `gh issue view <N>` | View issue |
| `gh issue create` | Create issue |
| `gh run list` | List Actions runs |
| `gh run view <id>` | View run details |
| `gh run view <id> --log` | View logs |
| `gh api` | Generic GitHub API |

---

## 📚 Sources
- Anthropic Code Review docs: https://code.claude.com/docs/en/code-review
- Official code-review plugin: https://github.com/anthropics/claude-code/tree/main/plugins/code-review
- `ykdojo/claude-code-tips` Tip 4 (git/gh), Tip 26 (PR reviews), Tip 29 (DevOps)
- Anthropic GitHub Actions: https://github.com/anthropics/claude-code-action
