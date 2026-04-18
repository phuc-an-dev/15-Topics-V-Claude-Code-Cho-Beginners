# Topic 11: Best Practices Manage Claude Memory

> **Nguồn**: Anthropic official docs, `shanraisshan/claude-code-best-practice`, Medium (Cuong Tham), dev.to (suede), shareuhack.com

---

## 3 Lớp Memory Trong Claude Ecosystem

Understand **3 lớp memory** tách biệt:

### Layer 1: **Chat Memory** (Web/Desktop)
- Cho Claude.ai users, **không phải Claude Code**
- Available on Free/Pro/Max plans (từ March 2026)
- Tự động: Claude nhớ preferences across chats

### Layer 2: **CLAUDE.md + Auto Memory** (Claude Code) 
- Cho developers dùng Claude Code CLI
- **Đây là layer quan trọng nhất cho bạn**
- 2 mechanisms: Manual (CLAUDE.md) + Auto (MEMORY.md)

### Layer 3: **Memory Tool API** (Developers building apps)
- Cho API users build own apps
- `type: memory_20250818`
- File-based, client-side controlled

**Focus của topic này**: **Layer 2** (CLAUDE.md + Auto Memory).

---

## CLAUDE.md - Manual Memory

### Bản chất
- File markdown **Claude đọc ở đầu mỗi session**
- Content inject vào system prompt
- Project constitution: rules **luôn apply**

### Locations (theo precedence từ CAO → THẤP)
```
1. ./.claude/CLAUDE.local.md  (gitignored, personal)
2. ./CLAUDE.md         (project, commit git)
3. ~/.claude/CLAUDE.md     (user global)
```

### Monorepo Loading Behavior (quan trọng)

> "Claude walks **UP** the directory tree, loads CLAUDE.md files ở startup."

```
/project/
 CLAUDE.md        ← Load ở startup
 frontend/
  CLAUDE.md      ← Load LAZY (khi Claude đọc files trong frontend/)
 backend/
  CLAUDE.md      ← Load LAZY
 api/
   CLAUDE.md      ← Load LAZY
```

**Rules**:
- **Ancestors**: Always load at startup (walks up)
- **Descendants**: Load lazily khi interact với files trong directory đó
- **Siblings**: Never load (frontend/ không load backend/CLAUDE.md)

---

## Nguyên Tắc Vàng CLAUDE.md

### Rule 1: **Keep It Minimal** (< 200 lines!)
> "Files over 200 lines consume more context and may **reduce adherence**." — Anthropic docs

### Rule 2: **Only Include Always-Relevant Rules**
Test: "Nếu xóa dòng này, Claude có làm sai không?" Không → xóa.

### Rule 3: **Specific and Concise**
> "The more specific and concise your instructions, the more consistently Claude follows them."

### Rule 4: **Use Import Syntax cho Long Content**
```markdown
# Project Context

See @README.md for project overview.
See @package.json for commands.

## Detailed Rules
- Git workflow: @docs/git-instructions.md
- Personal overrides: @~/.claude/my-prefs.md
```

### Rule 5: **Mark Important Rules**
```markdown
# IMPORTANT
- NEVER commit .env files
- YOU MUST run typecheck after edits
```

Emphasis keywords ("IMPORTANT", "YOU MUST", "NEVER") improve adherence.

---

## CLAUDE.md Template Siêu Gọn (< 100 lines)

```markdown
# Project: <Name>

## Stack
- <Main language + framework>
- <Database>
- <Package manager>

## Code Style (non-obvious)
- <Rule 1 that differs from defaults>
- <Rule 2 project-specific>

## Commands
- `<command 1>`: purpose
- `<command 2>`: purpose

## Conventions
- <Naming: camelCase/snake_case>
- <File organization>
- <Import style>

## IMPORTANT
- <Rule Claude MUST follow>
- <What NOT to do>

## Workflow
- Typecheck after changes
- Prefer single test runs (faster)
- Create draft PRs for review
```

---

## INCLUDE trong CLAUDE.md

| Include | Exclude |
|-----------|----------|
| Bash commands Claude can't guess | Things Claude figures out by reading code |
| Code style khác defaults | Standard language conventions |
| Testing instructions | Detailed API docs (link instead) |
| Branch naming, PR conventions | Frequently-changing info |
| Architectural decisions | Long tutorials |
| Env vars required | File-by-file descriptions |
| Non-obvious gotchas | Self-evident practices ("write clean code") |

---

## Auto Memory (MEMORY.md)

### Giới thiệu (v2.1.59+)
Claude tự maintain `MEMORY.md` ở `~/.claude/projects/<project>/memory/MEMORY.md`.

### Cách hoạt động
1. Claude observe corrections + preferences của bạn
2. Proactively write learnings vào MEMORY.md
3. Start new session: **First 200 lines** of MEMORY.md auto-load

### Khác biệt CLAUDE.md vs Auto Memory
| CLAUDE.md | Auto Memory |
|-----------|-------------|
| Manual, bạn viết | Automatic, Claude viết |
| "Your requirements" | "What Claude observed about you" |
| Edit với editor | Edit qua `/memory` hoặc tell Claude |
| Commit git được | Personal, local only |

### Toggle Auto Memory
```bash
/memory # Interactive management
```

Or settings:
```json
{ "autoMemoryEnabled": true }
```

### Browse Auto Memory
```bash
/memory
# Select "auto memory folder" to browse
# Files là plain markdown - read, edit, delete freely
```

---

## What Survives Compaction

### Survives 
- **Project-root CLAUDE.md**: Re-read from disk sau `/compact`, re-injected
- **Files trên disk**: HANDOFF.md, SPEC.md, docs/
- **Git commits**: `git log` luôn accessible

### Doesn't Survive 
- Conversation-only instructions ("always do X")
- Nested CLAUDE.md subdirectories (lazy reload, không auto-reinject)
- Clicked decisions không ghi xuống

**Lesson**: Nếu instruction quan trọng, PUT INTO CLAUDE.md, đừng chỉ nói trong chat.

---

## Hash `#` Shortcut

Gõ `#` trong Claude Code input → fast add to CLAUDE.md:
```
# Always use ES modules, not CommonJS
```

Claude prompt bạn chọn:
1. Project CLAUDE.md
2. User CLAUDE.md (~/.claude/)
3. Cancel

→ Faster than edit file manually.

---

## Subagent Memory (v2.1.33+, Feb 2026)

### Frontmatter `memory:` field
```markdown
---
name: code-reviewer
description: Reviews code for quality
tools: Read, Write, Edit, Bash
model: sonnet
memory: user # ← NEW
---

As you review code, update your agent memory with patterns,
conventions, and recurring issues you discover.
```

### Memory Scopes
- `memory: project` → `.claude/agent-memory/<agent-name>/MEMORY.md`
- `memory: user` → `~/.claude/agent-memory/<agent-name>/MEMORY.md`
- `memory: local` → `.claude/agent-memory/<agent-name>/MEMORY.md` (gitignored)
- `memory: none` (or absent) → No persistent memory

### Workflow
1. On startup: First 200 lines of agent's MEMORY.md loaded
2. During execution: Agent reads/writes its memory directory
3. Read/Write/Edit auto-enabled cho self-management

### Prompt để trigger memory usage
```markdown
Before starting, review your memory.
As you work, save architectural decisions and patterns.
```

---

## Advanced: External Memory Systems

### 1. **memory-mcp** (suede/claude-code-memory)
- MCP server cho persistent memory
- Silent capture (hooks sau mỗi response)
- 2-tier architecture:
 - **Tier 1**: CLAUDE.md (~150 lines, auto-updated)
 - **Tier 2**: `.memory/state.json` (unlimited, access qua MCP tools)
- Cost: ~$0.05-0.10/day (Haiku cho extraction)
- Install: `npm install -g claude-code-memory && memory-mcp setup`

### 2. **claude-mem** (thedotmack/claude-mem)
- Claude Code plugin
- Auto-captures session work
- Compress với agent-sdk
- Inject relevant context to future sessions
- Web viewer: http://localhost:37777
- Beta: "Endless Mode" (biomimetic memory)

### 3. **claude-obsidian** (AgriciDaniel)
- Uses Obsidian vault as memory
- `/wiki` mode: Karpathy's LLM Wiki pattern
- Autonomous knowledge base

---

## Pattern: Project Memory Structure

### Recommended folder layout
```
project/
 CLAUDE.md          # Project constitution (< 100 lines)
 .claude/
  settings.json
  agents/         # Subagent definitions
  skills/         # Custom skills
 docs/
  architecture.md     # Referenced via @docs/
  conventions.md
  decisions/        # ADRs
    001-use-postgres.md
    002-rest-vs-graphql.md
 SPEC.md           # Current feature spec (nếu có)
 HANDOFF.md          # Session continuation notes
```

### Reference trong CLAUDE.md
```markdown
# Quick Links
- Architecture: @docs/architecture.md
- Conventions: @docs/conventions.md
- Recent decisions: @docs/decisions/
```

---

## ️ claudeMdExcludes (Security)

Prevent untrusted CLAUDE.md from loading (vd từ git submodules):

```json
{
 "claudeMdExcludes": [
  "node_modules/**/CLAUDE.md",
  "vendor/**/CLAUDE.md",
  "untrusted-deps/**/CLAUDE.md"
 ]
}
```

Managed policy CLAUDE.md **cannot be excluded** → org-wide rules luôn apply.

---

## Pro Patterns

### Pattern 1: **Structured Initializer Session**
First session của project:
1. Create `CLAUDE.md` minimal
2. Create `docs/progress.md` checklist
3. Create `docs/architecture.md` overview
4. Mention `startup.sh` nếu có
5. Commit all

Future sessions load these artifacts → recover state instantly.

### Pattern 2: **Feature Checklist**
`docs/progress.md`:
```markdown
# Feature Progress

## In Progress
- [ ] User authentication
 - [x] Login endpoint
 - [ ] OAuth callback
 - [ ] Token refresh

## Completed
- [x] Database schema
- [x] Docker setup

## Next Up
- [ ] Payment integration
- [ ] Email notifications
```

**Rule**: Mark complete **only after end-to-end verification**, not after code written.

### Pattern 3: **End-of-Session Update**
Trước khi end session:
```
Before we stop, update docs/progress.md với:
- What we completed today
- Any architectural decisions made (add to docs/decisions/)
- What's the exact next step for next session
```

### Pattern 4: **Audit Memory Regularly**
Mỗi vài tuần:
```bash
find . -name "CLAUDE.md" # List all CLAUDE.md in project
```

Ask Claude:
```
Review this CLAUDE.md. Identify:
- Rules that are obvious/unnecessary
- Rules that are stale (tech has changed)
- Missing rules I've been repeating in conversations
Suggest concrete edits.
```

---

## Common Mistakes

### 1. **CLAUDE.md quá dài**
> "If your CLAUDE.md is too long, Claude **ignores half of it**." — Anthropic

Fix: Ruthlessly prune. Target < 200 lines.

### 2. **Task-specific info trong CLAUDE.md**
 "Currently working on payment integration"
 HANDOFF.md hoặc `docs/progress.md`

### 3. **Không Track What Claude Learns**
Auto Memory disabled → lose valuable learnings across sessions.
Fix: Enable auto memory, review `/memory` periodically.

### 4. **Committing Personal Preferences**
 "Always use tabs, I hate spaces" trong project CLAUDE.md
 `CLAUDE.local.md` (gitignored) hoặc `~/.claude/CLAUDE.md`

### 5. **Trust Conversation-only Instructions**
Nói "always do X" trong chat → `/compact` xảy ra → Claude quên.
Fix: Put vào CLAUDE.md immediately.

### 6. **Not Using Imports**
CLAUDE.md 500 lines.
Fix:
```markdown
See @docs/conventions.md for style rules.
See @docs/architecture.md for system design.
```

### 7. **Ignoring Nested CLAUDE.md Behavior**
Expect `backend/CLAUDE.md` load khi working trong `frontend/` → không load.
Understand lazy loading behavior of nested files.

### 8. **Not Reviewing Auto Memory**
`MEMORY.md` accumulate → stale info → bad suggestions.
Fix: `/memory` → browse → edit hoặc delete wrong learnings.

---

## Checklist Setup Memory

### Week 1: Foundation
- [ ] Create `./CLAUDE.md` < 100 lines
- [ ] Set up `~/.claude/CLAUDE.md` for personal prefs
- [ ] Enable Auto Memory (`autoMemoryEnabled: true`)
- [ ] Add `CLAUDE.local.md` to `.gitignore`

### Week 2: Organize
- [ ] Create `docs/architecture.md`
- [ ] Create `docs/conventions.md`
- [ ] Reference them với `@docs/` imports
- [ ] Setup Decision Records (`docs/decisions/`)

### Ongoing
- [ ] Review CLAUDE.md monthly
- [ ] Audit MEMORY.md periodically
- [ ] Update end of each major feature
- [ ] Use `#` shortcut cho quick additions

---

## Sources
- Anthropic Official: https://code.claude.com/docs/en/memory
- Anthropic Memory Tool API: https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool
- shanraisshan best practices: https://github.com/shanraisshan/claude-code-best-practice/blob/main/best-practice/claude-memory.md
- Subagent memory report: https://github.com/shanraisshan/claude-code-best-practice/blob/main/reports/claude-agent-memory.md
- Medium (Cuong Tham): https://medium.com/@codecentrevibe/claude-code-best-practices-memory-management-...
- dev.to (suede) - memory-mcp architecture: https://dev.to/suede/the-architecture-of-persistent-memory-for-claude-code-17d
- shareuhack guide: https://www.shareuhack.com/en/posts/claude-memory-feature-guide-2026
