# Topic 6: Best Practices Cấu Hình `.claude`

> **Nguồn**: Reddit r/ClaudeAI, Anthropic docs, `ykdojo/claude-code-tips`, `shanraisshan/claude-code-best-practice`

---

## 🗂️ Cấu Trúc Files `.claude` Quan Trọng

### Hierarchy (Thứ tự ưu tiên từ CAO → THẤP)
1. **Managed settings** (enterprise/org-level)
2. **Project settings**: `.claude/settings.json` (trong repo)
3. **User settings**: `~/.claude/settings.json` (global)
4. **Legacy**: `~/.claude.json` (nhiều settings chỉ hoạt động ở đây)

### Files chính trong `.claude`
```
~/.claude/
├── CLAUDE.md              ← Memory global (load mọi conversation)
├── settings.json          ← Config global
├── commands/              ← Slash commands tùy chỉnh
├── skills/                ← Skills tùy chỉnh
├── agents/                ← Subagents định nghĩa
└── projects/              ← Conversation history

./project/.claude/
├── CLAUDE.md              ← Memory project-specific
├── settings.json          ← Settings project (commit vào git)
├── commands/              ← Project slash commands
└── skills/                ← Project skills
```

---

## 📝 CLAUDE.md Best Practices

### ⭐ Nguyên tắc vàng: **KEEP IT SIMPLE**
- **Bắt đầu với KHÔNG có CLAUDE.md** (hoặc file rỗng)
- Chỉ thêm rule khi thấy phải lặp lại hướng dẫn với Claude nhiều lần
- **Review định kỳ** để xóa rules lỗi thời

### ⚠️ Anti-pattern: Over-specified CLAUDE.md
> "If your CLAUDE.md is too long, Claude **ignores half of it** because important rules get lost in the noise." — Anthropic official docs

**Fix**: Ruthlessly prune. Nếu Claude đã làm đúng mà không cần instruction → xóa nó hoặc chuyển sang hook.

### Cấu trúc CLAUDE.md hiệu quả (beginner-friendly)
```markdown
# Project: [Tên dự án]

## Tech Stack
- [Liệt kê ngắn gọn]

## Code Conventions
- [Rules quan trọng mà Claude hay quên]

## Commands
- `npm test`: chạy tests
- `npm run build`: build project

## Important
- [Rules BẮT BUỘC Claude phải tuân thủ]
```

### Pro tip: Dùng `#` shortcut
- Gõ `#` trong Claude Code → nhanh thêm vào CLAUDE.md
- Hoặc ask Claude: "Add this to CLAUDE.md" - nó biết edit ở đâu

### Periodic Review với skill
- Dùng skill `review-claudemd` (trong dx plugin) để Claude phân tích conversations gần đây và suggest cải thiện CLAUDE.md

---

## ⚙️ settings.json - Các Config Quan Trọng

### Cấu hình cơ bản mọi người nên có
```json
{
  "permissions": {
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "Read(./config/credentials.json)"
    ],
    "allow": [
      "Read(~/.claude)",
      "Bash(git pull:*)",
      "Bash(git status:*)",
      "Bash(git log:*)"
    ]
  },
  "env": {
    "DISABLE_AUTOUPDATER": "1",
    "ENABLE_TOOL_SEARCH": "true"
  },
  "attribution": {
    "commit": "",
    "pr": ""
  }
}
```

### Giải thích từng setting

| Setting | Lý do dùng |
|---------|-----------|
| `permissions.deny` | **QUAN TRỌNG** - Chặn Claude đọc `.env`, secrets |
| `permissions.allow` | Whitelist commands không cần hỏi, giảm friction |
| `DISABLE_AUTOUPDATER` | Giữ version ổn định, tránh break config/patches |
| `ENABLE_TOOL_SEARCH` | **Tiết kiệm tokens** - lazy-load MCP tools khi cần |
| `attribution` (empty) | Xóa "Co-Authored-By Claude" khỏi commits/PRs |

### Security Warning ⚠️
Theo nhiều reports trên Reddit:
- **Deny rules đôi khi bị bỏ qua** (Claude vẫn đọc `.env` dù đã deny)
- → Không nên 100% tin vào deny rules, nên dùng kèm `.gitignore`
- Dùng tool `cc-safe` để audit approved commands: `npx cc-safe .`

---

## 🎨 Customization cho Developer Experience

### 1. Status Line tùy chỉnh
Hiển thị token usage, git branch, model ở bottom:
```
Opus 4.5 | 📁my-project | 🔀main | ██░░░░░░░░ 18% of 200k tokens
```
→ Dùng `scripts/context-bar.sh` từ `ykdojo/claude-code-tips`

### 2. Terminal Aliases
Thêm vào `~/.zshrc` hoặc `~/.bashrc`:
```bash
alias c='claude'
alias ch='claude --chrome'        # với Chrome integration
alias cs='claude --dangerously-skip-permissions'  # CHỈ DÙNG TRONG CONTAINER!
```

### 3. Fork session shortcut
```bash
claude() {
  local args=()
  for arg in "$@"; do
    if [[ "$arg" == "--fs" ]]; then
      args+=("--fork-session")
    else
      args+=("$arg")
    fi
  done
  command claude "${args[@]}"
}
```
Dùng: `c -c --fs` để fork conversation gần nhất.

---

## 🔌 MCP Configuration

### Project-scoped `.mcp.json`
Commit vào git, share cho team:
```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp@latest"]
    }
  }
}
```

### Tips
- **Lazy-load MCP tools**: `ENABLE_TOOL_SEARCH=true` → tiết kiệm ~10k tokens
- Install Playwright MCP (recommended): `claude mcp add -s user playwright npx @playwright/mcp@latest`

---

## 🪝 Hooks - Deterministic Actions

### Khi nào dùng hooks?
- Actions **phải xảy ra mỗi lần** (không exception)
- Ví dụ: run linter sau edit, block writes vào `migrations/`
- **Khác với CLAUDE.md**: CLAUDE.md = advisory (Claude có thể bỏ qua), hooks = deterministic

### Ví dụ hook đơn giản
```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/scripts/check-context.sh"
          }
        ]
      }
    ]
  }
}
```

### Pro tip
Hỏi Claude viết hook cho bạn:
> "Write a hook that runs eslint after every file edit"
> "Write a hook that blocks writes to the migrations folder"

---

## 📌 Checklist Setup cho Beginners

### Setup lần đầu (theo thứ tự)
- [ ] 1. Tạo `~/.claude/settings.json` với `permissions.deny` cho `.env`, secrets
- [ ] 2. Bật `ENABLE_TOOL_SEARCH` trong env
- [ ] 3. Thêm aliases `c`, `ch` vào shell config
- [ ] 4. KHÔNG tạo CLAUDE.md vội - chờ nhu cầu thực sự
- [ ] 5. Install Playwright MCP (hữu ích cho web research)
- [ ] 6. Chạy `cc-safe .` để check settings không có risky commands
- [ ] 7. Disable attribution (nếu không muốn "Co-Authored-By" trong commits)

### Quick Setup Script (one-liner)
```bash
bash <(curl -s https://raw.githubusercontent.com/ykdojo/claude-code-tips/main/scripts/setup.sh)
```
→ Script interactive, cho chọn skip từng item.

---

## 🧭 Nơi Đặt Cái Gì Ở Đâu?

| Settings | Global (`~/.claude/`) | Project (`./.claude/`) |
|----------|----------------------|------------------------|
| `CLAUDE.md` | Personal preferences, style | Project-specific context |
| Permissions | Default deny cho secrets | Project-specific allows |
| Skills | Tools bạn dùng mọi nơi | Project-specific workflows |
| `.mcp.json` | ❌ (dùng `~/.claude.json`) | ✅ Share với team qua git |
| Aliases | Shell config, không phải Claude | — |

---

## 🚨 Common Mistakes

1. **CLAUDE.md quá dài** → Claude bỏ qua rules quan trọng
2. **Không deny `.env`** → Claude có thể đọc secrets vào context
3. **Enable auto-updater nhưng có patches** → Mỗi lần update phá hết customization
4. **Commit `settings.json` với personal info** → Leak API keys, paths
5. **Dùng legacy `~/.claude.json` cho mọi thứ** → Khó maintain

---

## 📚 Resources
- Official: https://code.claude.com/docs/en/settings
- `shanraisshan/claude-code-best-practice` - Comprehensive settings guide
- `ykdojo/claude-code-tips` - Tip 0, 7, 15, 25, 30, 33
- ClaudeLog: https://claudelog.com/configuration/ - Full 60+ settings reference
- r/ClaudeAI weekly discussions
