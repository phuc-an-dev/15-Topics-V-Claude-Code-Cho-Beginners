# Topic 4: Best Claude Skills Cho Developer (General)

> **Nguồn**: `travisvn/awesome-claude-skills`, `karanb192/awesome-claude-skills`, `ComposioHQ/awesome-claude-skills`, `VoltAgent/awesome-agent-skills`, Anthropic official skills repo

---

## Claude Skills Là Gì?

**Skills = folders chứa `SKILL.md`** giúp Claude làm task chuyên sâu.

### Đặc điểm quan trọng
- **Progressive disclosure**: Chỉ ~30-100 tokens metadata scan. Full content load (< 5KB) khi activate.
- **Auto-invoke**: Claude tự load khi relevant, hoặc gọi manual `/skill-name`
- **Composable**: Nhiều skills hoạt động cùng nhau (vd TDD + debugging + git-worktrees)
- **Cross-platform**: Cùng format cho Claude.ai, Claude Code CLI, Claude API

### Cấu trúc cơ bản
```markdown
---
name: my-skill
description: Brief description for discovery (keep concise)
---
# Instructions for Claude
[Detailed steps, examples, references]
```

Đặt ở: `~/.claude/skills/<skill-name>/SKILL.md` (global) hoặc `./.claude/skills/<skill-name>/` (project)

️ **Security**: Skills có thể execute code → **chỉ install từ trusted sources**.

---

## TOP Skills Bundles Cho Developer (must-have)

### 1. **obra/superpowers** (Most recommended)
**Repo**: https://github.com/obra/superpowers

- **20+ battle-tested skills** core cho Claude Code
- Commands: `/brainstorm`, `/write-plan`, `/execute-plan`
- Tool `skills-search` để discover skills
- Include: TDD, debugging, collaboration patterns

**Install**:
```bash
/plugin marketplace add obra/superpowers-marketplace
```

### 2. **Jeffallan/claude-skills** (1.5k )
**Repo**: https://github.com/Jeffallan/claude-skills

**66 specialized skills cho full-stack devs** across 12 categories:
- Languages (Python, TypeScript, Rust, Go, ...)
- Backend frameworks (NestJS, Django, Rails, Spring Boot)
- Frontend (React, Vue, Svelte)
- Infrastructure (K8s, Docker, Terraform)
- APIs, Testing, DevOps, Security, Data/ML

**Auto-activate based on request**:
- "Implement JWT auth in NestJS" → NestJS Expert
- "Build React Server Components" → React Expert
- "Debug memory leak" → Debugging Wizard

### 3. **karanb192/awesome-claude-skills**
**Repo**: https://github.com/karanb192/awesome-claude-skills

**50+ verified skills** - definitive collection. Tốt để browse theo category.

### 4. **VoltAgent/awesome-agent-skills**
**Repo**: https://github.com/VoltAgent/awesome-agent-skills

**1000+ skills** từ official teams (Anthropic, Stripe, Cloudflare, Netlify, Trail of Bits, Sentry, Figma, Hugging Face, Vercel). Multi-platform (Claude Code, Cursor, Codex, Gemini CLI).

---

## ️ Top Individual Skills Cho Developer

### A. **Testing & Quality**

#### 1. **test-driven-development** (from obra/superpowers)
- Use **before writing any implementation code**
- Enforces: write tests first → fail → implement → pass
- Prevents Claude's tendency to code first, test later

#### 2. **systematic-debugging**
- Use khi gặp bug, test failure, unexpected behavior
- Structured approach: hypothesis → evidence → root cause
- Prevents band-aid fixes

#### 3. **Webapp Testing** (ComposioHQ)
- Dùng Playwright để test local web apps
- Verify frontend functionality, debug UI
- Capture screenshots tự động

---

### B. **Git & Version Control**

#### 4. **using-git-worktrees** (from obra/superpowers)
- Tạo isolated git worktrees với smart directory selection
- Safety verification trước khi tạo
- **Cực hữu ích cho parallel Claude sessions**

#### 5. **finishing-a-development-branch**
- Guide completing development work
- Present clear options (merge vs rebase vs PR)
- Handle chosen workflow automatically

#### 6. **Changelog Generator**
- Auto-create user-facing changelogs từ git commits
- Transform technical commits → customer-friendly release notes

---

### C. **Architecture & Design**

#### 7. **software-architecture**
- Implements Clean Architecture, SOLID principles
- Comprehensive design best practices
- Use when starting new service/module

#### 8. **subagent-driven-development**
- Dispatch independent subagents cho individual tasks
- Code review checkpoints between iterations
- Rapid, controlled development

---

### D. **Security**

#### 9. **owasp-security**
- OWASP Top 10:2025, ASVS 5.0, Agentic AI security (2026)
- Code review checklists
- Language-specific quirks cho 20+ languages

#### 10. **Trail of Bits Security Skills** 
**Repo**: https://github.com/trailofbits/claude-skills (1.3k )
- Static analysis với CodeQL/Semgrep
- Variant analysis, code auditing, fix verification
- Industry-leading security skills

#### 11. **VibeSec-Skill**
- Write secure code, prevent common vulnerabilities

#### 12. **varlock-claude-skill**
- Secure environment variable management
- Secrets KHÔNG BAO GIỜ xuất hiện trong sessions, logs, git commits

#### 13. **sanitize**
- Detect và redact PII (15 categories: SSN, credit cards, API keys, etc.)
- Zero dependencies, all local

---

### E. **Document Processing (Anthropic official)**

#### 14. **anthropics/docx** + **anthropics/pptx** + **anthropics/xlsx** + **anthropics/pdf**
- Create/edit/analyze Word, PowerPoint, Excel, PDF
- Official, maintained by Anthropic
- Essential cho devs làm reports, documentation

---

### F. **Developer Experience & Productivity**

#### 15. **artifacts-builder**
- Create elaborate multi-component claude.ai HTML artifacts
- React, Tailwind CSS, shadcn/ui integration

#### 16. **Claude Code Terminal Title**
- Dynamic title describing current work
- Không lose track khi có nhiều terminal windows

#### 17. **MCP Builder**
- Guide creating high-quality MCP servers
- Python hoặc TypeScript
- Meta-skill để extend Claude itself

#### 18. **Connect** (Composio)
- Connect Claude với 1000+ apps (Gmail, Slack, GitHub, Notion)
- Send emails, create issues, post messages
- Handle auth automatically

---

### G. **Platform-Specific**

#### 19. **aws-skills**
- AWS development với CDK best practices
- Cost optimization MCP servers
- Serverless/event-driven patterns

#### 20. **iOS Simulator**
- Interact với iOS Simulator cho testing/debugging

#### 21. **antonbabenko/terraform-skill**
- Terraform IaC best practices

---

## Recommended Starter Pack cho Beginners

### Must-Have (cài ngay tuần đầu)
1. **obra/superpowers** - Core development skills
2. **Anthropic official**: docx, pptx, xlsx, pdf (docs work)
3. **test-driven-development** - Prevent bad habits
4. **systematic-debugging** - Structured bug fixing
5. **using-git-worktrees** - Parallel work

### Nice-to-Have (tùy stack)
6. **software-architecture** - Khi design mới
7. **owasp-security** - Production code
8. **Changelog Generator** - Nếu maintain releases
9. Stack-specific từ `Jeffallan/claude-skills` (NestJS/React/Spring/etc.)

### Advanced (sau khi comfortable)
10. **subagent-driven-development** - Cho complex tasks
11. **MCP Builder** - Build own integrations
12. **Trail of Bits skills** - Security audits

---

## How to Install Skills

### Method 1: Plugin Marketplace (easiest)
```bash
# Add marketplace
/plugin marketplace add <org>/<repo>

# Install specific plugin
/plugin install <plugin-name>@<org>
```

### Method 2: Manual Install
```bash
# Global
cd ~/.claude/skills
git clone https://github.com/obra/superpowers.git

# Project-specific
cd ./.claude/skills
git clone ...
```

### Method 3: Copy SKILL.md directly
- Copy file vào `~/.claude/skills/<name>/SKILL.md`
- Add any `scripts/`, `references/`, `assets/` folders nếu skill cần

### Update Skills
```bash
cd ~/.claude/skills/<skill-name>
git pull origin main
```

---

## Best Practices Dùng Skills

### 1. **Start Small**
- Cài 3-5 skills đầu tiên, dùng cho comfortable
- Không cài 50 skills một lúc → overwhelmed

### 2. **Review SKILL.md Before Install**
- Skills có thể execute code
- Đọc qua trước khi install, đặc biệt script sections

### 3. **Let Claude Auto-Invoke**
- Skills được trigger bởi description match
- Không cần gọi manually hầu hết cases
- Exception: slash-command style skills (`/fix-issue`)

### 4. **Compose Skills**
Claude tự kết hợp khi relevant:
> "Implement OAuth với security review" → TDD + software-architecture + owasp-security tự load

### 5. **Create Own Skills cho Repetitive Tasks**
- Nếu lặp cùng instructions 3+ lần → làm skill
- Dùng `skill-creator` skill để guide

### 6. **Skills vs CLAUDE.md vs Slash Commands**

| Feature | Khi nào dùng |
|---------|-------------|
| **CLAUDE.md** | Context luôn-đúng, mọi session cần |
| **Skills** | Domain knowledge occasional, workflows |
| **Slash commands** | User-triggered precise actions |
| **Subagents** | Isolated context cho specific tasks |
| **MCP servers** | External APIs/data sources |

---

## Key Repos để Bookmark

| Repo | Stars | Nội dung |
|------|-------|---------|
| [anthropics/skills](https://github.com/anthropics/skills) | 37.5k | Official Agent Skills |
| [obra/superpowers](https://github.com/obra/superpowers) | High | Core dev skills |
| [Jeffallan/claude-skills](https://github.com/Jeffallan/claude-skills) | 1.5k | 66 full-stack skills |
| [karanb192/awesome-claude-skills](https://github.com/karanb192/awesome-claude-skills) | - | 50+ verified |
| [VoltAgent/awesome-agent-skills](https://github.com/VoltAgent/awesome-agent-skills) | - | 1000+ multi-platform |
| [travisvn/awesome-claude-skills](https://github.com/travisvn/awesome-claude-skills) | - | Curated w/ analysis |
| [ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills) | - | Practical productivity |
| [jqueryscript/awesome-claude-code](https://github.com/jqueryscript/awesome-claude-code) | - | Ecosystem overview |

---

## ️ Security Warning

> "Skills can execute **arbitrary code** in Claude's environment. Only install skills from **trusted sources**."

**Red flags**:
- Unknown author with no history
- Skill asks to disable permissions
- Scripts fetch from random URLs
- No documentation on what skills do

**Safe practice**:
- Stick to official (anthropics/*) và well-known community repos (obra, Jeffallan, etc.)
- Review SKILL.md + any scripts trước khi install
- Run `npx cc-safe .` định kỳ để audit

---

## Resources
- Skills Announcement: https://www.anthropic.com/news/skills
- Skills Explained: https://docs.claude.com/en/docs/agents-and-tools/skills
- Discovery sites: agentskill.sh, officialskills.sh
- Simon Willison analysis: "Claude Skills are awesome, maybe a bigger deal than MCP"
