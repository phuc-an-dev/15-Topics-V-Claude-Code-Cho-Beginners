# TỔNG HỢP: 15 Topics Về Claude Code Cho Beginners

> **Nguồn**: Reddit r/ClaudeAI, r/ClaudeCode, GitHub repos (6.6k+ stars), Anthropic official docs
> **Ngày**: April 2026
> **Dành cho**: Beginners dùng Claude Code CLI + VS Code/Antigravity extension

---

## ️ Danh Sách Files Chi Tiết

| # | File | Nội dung chính |
|---|------|----------------|
| 1 | [01-best-practices-using-claude.md](./01-best-practices-using-claude.md) | Best practices using Claude (general) |
| 2 | [02-best-skills-react-typescript.md](./02-best-skills-react-typescript.md) | Skills cho React + TypeScript dev |
| 3 | [03-best-skills-spring-boot.md](./03-best-skills-spring-boot.md) | Skills cho Spring Boot dev |
| 4 | [04-best-skills-developer.md](./04-best-skills-developer.md) | Skills chung cho developers |
| 5 | [05-khong-bi-token-limit.md](./05-khong-bi-token-limit.md) | Tránh token limit |
| 6 | [06-best-practice-cau-hinh-claude.md](./06-best-practice-cau-hinh-claude.md) | Cấu hình `.claude` |
| 7 | [07-best-concepts.md](./07-best-concepts.md) | Concepts quan trọng (TDD, etc.) |
| 8 | [08-claude-with-github.md](./08-claude-with-github.md) | Claude với GitHub |
| 9 | [09-claude-with-obsidian.md](./09-claude-with-obsidian.md) | Claude với Obsidian |
| 10 | [10-best-practice-using-agent.md](./10-best-practice-using-agent.md) | Best practice với Agents/Subagents |
| 11 | [11-manage-claude-memory.md](./11-manage-claude-memory.md) | Quản lý Claude memory |
| 12 | [12-session-memory-preservation.md](./12-session-memory-preservation.md) | Giữ context giữa sessions |
| 13 | [13-best-hooks-for-developer.md](./13-best-hooks-for-developer.md) | Best hooks cho developer |
| 14 | [14-plan-mode-best-practice.md](./14-plan-mode-best-practice.md) | Plan Mode best practices |
| 15 | [15-mistakes-anti-patterns.md](./15-mistakes-anti-patterns.md) | 15 mistakes cần tránh |

---

## TOP 10 Key Insights (Đọc Ngay!)

### 1. **Context is your fundamental constraint**
> "Most best practices are based on one constraint: Claude's context window fills up fast, and performance degrades as it fills." — Anthropic

→ Tất cả mọi practice khác đều derive từ nguyên tắc này.

### 2. **`/clear` là đòn bẩy mạnh nhất**
> "The single most effective lever for both quality and cost."

**Rule**: `/clear` giữa các tasks. Sau 2 correction failures → `/clear` + better prompt.

### 3. **Explore → Plan → Code → Commit** (4 phases)
Không let Claude jump straight to coding - sẽ solve **wrong problem**.

### 4. **CLAUDE.md: Keep It Short (< 200 lines)**
> "If your CLAUDE.md is too long, Claude **ignores half of it**."

### 5. **Give Claude a way to verify**
> "The single highest-leverage thing you can do." — Anthropic

Tests, screenshots, linter, typecheck. Không verify được → don't ship.

### 6. **Specific prompts beat vague ones**
```
 "fix the login bug"
 "users report login fails after session timeout.
  Check auth flow in src/auth/, especially token refresh.
  Write failing test first."
```

### 7. **Subagents = separate context windows**
Delegate investigation/research → main conversation stays clean.

### 8. **2-Strike Rule for CLAUDE.md**
Correct Claude **once** → fine.
**Twice** same issue → add to CLAUDE.md.
**3+ times** → convert to Skill or Hook.

### 9. **Tests First (TDD) với Claude**
Claude defaults to implementation-first. Explicitly prompt:
> "Write FAILING test for X. Do NOT write implementation yet."

### 10. **Hooks = Deterministic, CLAUDE.md = Advisory**
Rules bắt buộc phải happen → Hooks.
Guidelines, conventions → CLAUDE.md.

---

## TOP 10 Mistakes Để Tránh

| # | Mistake | Fix |
|---|---------|-----|
| 1 | CLAUDE.md quá dài | Prune to < 200 lines |
| 2 | Kitchen sink sessions (nhiều tasks trong 1 session) | `/clear` giữa tasks |
| 3 | Correct > 2 lần same issue | `/clear` + better prompt |
| 4 | Trust Claude mà không verify | Always provide verification |
| 5 | "Investigate X" (vague) | Scope hẹp + subagents |
| 6 | Skip Plan Mode cho tasks phức tạp | Use Plan Mode cho multi-file |
| 7 | Vague prompts | Specific file refs + constraints |
| 8 | `--dangerously-skip-permissions` trên host | Chỉ trong containers |
| 9 | Không review Claude's code | Ask "why?" + visual diff |
| 10 | Không deny `.env` trong settings | `permissions.deny` + `.gitignore` |

---

## ️ Setup Essentials (Cho Beginners)

### Day 1 Setup (30 phút)
```bash
# 1. Install gh CLI (required cho GitHub workflows)
brew install gh && gh auth login

# 2. Setup terminal aliases (~/.zshrc or ~/.bashrc)
alias c='claude'
alias ch='claude --chrome'

# 3. Create ~/.claude/settings.json
```

**Minimal `~/.claude/settings.json`**:
```json
{
 "permissions": {
  "deny": ["Read(./.env)", "Read(./.env.*)", "Read(./secrets/**)"]
 },
 "env": {
  "ENABLE_TOOL_SEARCH": "true"
 },
 "attribution": {
  "commit": "",
  "pr": ""
 }
}
```

### Week 1: Foundation Skills
```bash
# Install essential plugins
/plugin marketplace add obra/superpowers-marketplace
/plugin install superpowers@superpowers-marketplace

# Or quick all-in-one setup
bash <(curl -s https://raw.githubusercontent.com/ykdojo/claude-code-tips/main/scripts/setup.sh)
```

### Month 1: Advanced
- Custom hooks (auto-format, block dangerous commands, notifications)
- Custom skills cho workflows lặp đi lặp lại
- Subagents cho domain-specific tasks

---

## Quick Reference Tables

### Essential Slash Commands
| Command | Purpose |
|---------|---------|
| `/clear` | Start fresh session |
| `/compact` | Compress context |
| `/plan` or `Shift+Tab` | Plan Mode |
| `/usage` | Check rate limits |
| `/stats` | Usage stats |
| `/hooks` | Browse configured hooks |
| `/memory` | Manage auto memory |
| `/rewind` or `Esc+Esc` | Undo to checkpoint |
| `/btw` | Side question (không save context) |
| `/release-notes` | New features |

### Input Box Shortcuts
| Shortcut | Action |
|----------|--------|
| `Ctrl+A` | Start of line |
| `Ctrl+E` | End of line |
| `Ctrl+W` | Delete previous word |
| `Ctrl+G` | Open prompt in external editor |
| `\` + Enter | New line |
| `Esc` | Stop Claude mid-action |

### Khi Nào Dùng Gì?
| Situation | Tool |
|-----------|------|
| Context full, mid-task | `/compact <focus>` |
| Starting new task | `/clear` |
| Complex multi-file task | Plan Mode (Shift+Tab) |
| Investigation needs many files | Subagent |
| Workflow lặp lại 3+ lần | Skill hoặc Hook |
| Rule phải happen mọi lần | Hook |
| Context always-relevant | CLAUDE.md |
| Task-specific notes | HANDOFF.md |
| Feature spec | SPEC.md |

---

## Top GitHub Repos (Bookmark)

### Essential
- **[ykdojo/claude-code-tips](https://github.com/ykdojo/claude-code-tips)** (6.6k ) - 45 tips cực hay
- **[anthropics/skills](https://github.com/anthropics/skills)** (37.5k ) - Official skills
- **[anthropics/claude-code](https://github.com/anthropics/claude-code)** (55k ) - Main repo

### Skills Collections
- **[obra/superpowers](https://github.com/obra/superpowers)** - Core dev skills
- **[Jeffallan/claude-skills](https://github.com/Jeffallan/claude-skills)** (1.5k ) - 66 full-stack skills
- **[karanb192/awesome-claude-skills](https://github.com/karanb192/awesome-claude-skills)** - 50+ verified
- **[VoltAgent/awesome-agent-skills](https://github.com/VoltAgent/awesome-agent-skills)** - 1000+ skills
- **[travisvn/awesome-claude-skills](https://github.com/travisvn/awesome-claude-skills)** - Curated w/ analysis

### Stack-Specific
- **React + TS**: `mattpocock/skills`, `spartan-ai-toolkit`
- **Spring Boot**: `piomin/claude-ai-spring-boot`, `a-pavithraa/springboot-skills-marketplace`

### Tools
- **Hooks**: `disler/claude-code-hooks-mastery`
- **Safety**: `ykdojo/cc-safe` - Audit risky commands
- **Subagents**: `VoltAgent/awesome-claude-code-subagents` - 100+ specialized
- **Memory**: `suede/memory-mcp`, `thedotmack/claude-mem`

### Obsidian Integration
- **[iansinnott/obsidian-claude-code-mcp](https://github.com/iansinnott/obsidian-claude-code-mcp)** - Native plugin
- **[MarkusPfundstein/mcp-obsidian](https://github.com/MarkusPfundstein/mcp-obsidian)** - Python MCP server
- **[AgriciDaniel/claude-obsidian](https://github.com/AgriciDaniel/claude-obsidian)** - LLM Wiki pattern

---

## Mindset Shifts Cho Beginners

### Từ "Chatbot" → "Collaborator"
Claude Code không phải chatbot trả lời câu hỏi. Là **collaborator** bạn architect:
- Write CLAUDE.md = teach conventions
- Create Skills = extend capabilities
- Configure Hooks = enforce rules
- Use Subagents = delegate specialized work

### Từ "One-shot prompt" → "Tight feedback loops"
- Correct early, correct often
- After 2 fails → `/clear` + better prompt
- Verify output continuously

### Từ "Act first" → "Explore first"
- Plan Mode cho complex tasks
- Interview me pattern cho features lớn
- SPEC.md trước implementation

### Từ "Do everything yourself" → "Automation of automation"
- Task lặp 3+ lần → automate
- CLAUDE.md → Skills → Hooks → Scripts
- Invest trong workflow (pays compound returns)

### Từ "Fear the unknown" → "Be brave"
- Claude Code makes unfamiliar tech accessible
- Iteratively explore, experiment
- Control the zoom level (vibe coding ↔ deep dive)

---

## Learning Path Cho Beginners

### Week 1: Basics
- [ ] Install Claude Code + gh CLI
- [ ] Setup `~/.claude/settings.json` minimal
- [ ] Learn essential slash commands
- [ ] Practice: Do 3 small tasks với Claude (fix typo, add feature, write tests)
- [ ] Read Anthropic Best Practices doc

### Week 2: Workflow
- [ ] Create first CLAUDE.md cho your project (< 100 lines)
- [ ] Try Plan Mode cho 1 complex task
- [ ] Learn `/clear` vs `/compact` vs `/rewind`
- [ ] Setup status line với token indicator
- [ ] Install first skill (TDD or debugging)

### Week 3: Customization
- [ ] Install obra/superpowers
- [ ] Install stack-specific skills (React/TS hoặc Spring Boot)
- [ ] Create first HANDOFF.md
- [ ] Setup 3 essential hooks (format, block dangerous, notify)
- [ ] Try subagents for investigation task

### Week 4: Advanced
- [ ] Setup MCP servers (Playwright, Obsidian nếu dùng)
- [ ] Custom skill cho workflow của bạn
- [ ] Custom hook enforce project conventions
- [ ] Git worktrees cho parallel sessions
- [ ] Review all your CLAUDE.md files - prune mercilessly

### Month 2: Expert
- [ ] Multi-agent workflows (Writer/Reviewer pattern)
- [ ] Custom TDD pipeline với subagents
- [ ] Container-based Claude Code cho risky tasks
- [ ] Automation of automation - find & automate 5 repetitive workflows
- [ ] Contribute back: share tips on r/ClaudeAI, open issues to Anthropic

---

## External Resources

### Official
- **Docs**: https://code.claude.com/docs/en/overview
- **Best Practices**: https://code.claude.com/docs/en/best-practices
- **Blog**: https://claude.com/blog/
- **Changelog**: `/release-notes` in Claude Code

### Community
- **Reddit**: r/ClaudeAI, r/ClaudeCode
- **X/Twitter**: @adocomplete (DevRel Anthropic), @AnthropicAI
- **YouTube**: Multiple tutorials, "Claude Code Masterclass"
- **Newsletter**: "Agentic Coding with Discipline" (Substack)

### Learning
- **Anthropic Academy**: 13 free courses (Claude Code, API, MCP, Skills)
- **Steve Kinney Course**: TDD with Claude Code
- **claudelog.com**: Comprehensive reference
- **claudefa.st**: Guides, optimization

---

## Final Wisdom

### Top 3 Principles từ Research

#### 1. **"Context is like milk - fresh and condensed is best"**
Đừng ngại `/clear`. Đừng sợ mất context. Fresh context + good HANDOFF.md = better than polluted long session.

#### 2. **"Developer skills matter more than ever"**
Claude Code không replace skills - **amplify** chúng.
- Software engineering principles still apply
- Debugging, architecture, problem breakdown skills = exponential với Claude

#### 3. **"Billion token rule > 10,000 hour rule"**
Cách tốt nhất học Claude = dùng Claude nhiều. Consume tokens, build intuition, iterate.

### Lời Khuyên Cuối
> "The best way to get better at using Claude Code is **by using it**." — ykdojo

> "Start with what matches the work you actually do and let the **gaps tell you what to add next**." — welcomedeveloper

> "Treat Claude as a **system you architect**, not a chatbot you prompt." — Principal Engineer @ Cars24

---

## Notes Riêng Cho Bạn (Vietnamese Developer)

Dựa trên context bạn dùng Claude Code CLI + VS Code/Antigravity extension:

### Ưu tiên cao
1. **Topic 6** (cấu hình `.claude`) + **Topic 15** (mistakes) - đọc ngay để tránh lỗi
2. **Topic 5** (token limit) + **Topic 11** (memory) - quan trọng để không waste rate limit
3. **Topic 14** (Plan Mode) + **Topic 7** (concepts) - đổi mindset từ "chat" sang "architect"
4. **Topic 3** hoặc **Topic 2** (skills theo stack của bạn)

### Practical First Steps
1. Copy minimal `settings.json` từ section "Setup Essentials" ở trên
2. Create CLAUDE.md ngắn (<100 lines) cho 1 project nhỏ bạn đang làm
3. Try Plan Mode cho feature tiếp theo
4. Học 5 slash commands: `/clear`, `/compact`, `/plan`, `/usage`, `/rewind`
5. Install **obra/superpowers** để có TDD + debugging skills foundation

### Tránh làm quá sớm
- Custom hooks phức tạp (wait Week 3+)
- Multi-agent orchestration (wait Month 2+)
- Agent teams (tốn ~7x tokens - wait until comfortable)
- Patching system prompt (rủi ro cao cho beginners)

### Khi gặp vấn đề
- Claude "ngu đi" mid-session → `/clear` và restart
- Rate limit hết → switch `/model haiku` hoặc `/model sonnet`
- CLAUDE.md bị ignore → chắc chắn nó < 200 lines
- Claude xóa file không mong muốn → check `cc-safe` audit
- Context rot trên monorepo → dùng nested CLAUDE.md hoặc refactor

---

**Good luck on your Claude Code journey! **

*Bộ research này được tạo bằng Claude Opus 4.7 qua web search trên Reddit, GitHub, và Anthropic official docs ngày 18/04/2026.*
