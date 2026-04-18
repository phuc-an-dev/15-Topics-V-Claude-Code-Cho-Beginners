# Topic 1: Best Practices Using Claude (Claude Code)

> **Nguồn chính**: Reddit r/ClaudeAI, r/ClaudeCode, GitHub `ykdojo/claude-code-tips` (6.6k stars), Anthropic official docs

---

## Key Principles (Những nguyên tắc cốt lõi)

### 1. **Context is like milk - fresh and condensed is best**
- Claude hoạt động tốt nhất khi context **ngắn và mới**
- Khi conversation quá dài → performance giảm
- **→ Start new conversation** cho mỗi topic mới hoặc khi thấy Claude "ngu đi"

### 2. **Break down large problems** (Chia nhỏ vấn đề)
- Đây là kỹ năng **quan trọng nhất** (theo ykdojo - 6.6k stars repo)
- Thay vì đi A → B, hãy đi A → A1 → A2 → A3 → B
- Nếu Claude không one-shot được task, hãy bảo nó **break down thành smaller issues**
- Software engineering skills của bạn vẫn cực kỳ quan trọng khi dùng Claude

### 3. **Separate planning from implementation** (Tách lập kế hoạch khỏi code)
- **Dùng Plan Mode** (Shift+Tab) để Claude research trước khi code
- Workflow 4 phases (Anthropic official):
 1. **Explore**: Plan Mode - Claude đọc files, trả lời câu hỏi, không sửa gì
 2. **Plan**: Yêu cầu tạo implementation plan chi tiết
 3. **Edit plan**: Ctrl+G để mở plan trong editor, sửa trực tiếp
 4. **Execute**: Normal Mode, cho Claude code theo plan

### 4. **Complete the write-test cycle** (Hoàn thành chu trình write-test)
- Muốn Claude làm autonomous task → phải cho nó **cách verify kết quả**
- Ví dụ: write code → run tests → check output → fix → repeat
- TDD hoạt động rất tốt với Claude Code

### 5. **Invest in your own workflow** (Đầu tư vào workflow cá nhân)
- Customize `CLAUDE.md` của mình
- Học các slash commands (`/compact`, `/clear`, `/plan`, `/usage`)
- Setup terminal aliases (vd: `c` = `claude`)

---

## Practical Tips cho Beginners

### A. Essential Slash Commands (học thuộc ngay)
| Command | Mục đích |
|---------|----------|
| `/clear` | Xóa conversation, bắt đầu mới |
| `/compact` | Nén context lại để tiết kiệm tokens |
| `/plan` hoặc `Shift+Tab` | Vào Plan Mode |
| `/usage` | Check rate limits còn lại |
| `/copy` | Copy last response ra clipboard |
| `/mcp` | Quản lý MCP servers |
| `/stats` | Xem thống kê usage |
| `/release-notes` | Xem features mới |

### B. Input Box Shortcuts (điều hướng nhanh)
- `Ctrl+A`: Về đầu dòng
- `Ctrl+E`: Về cuối dòng
- `Ctrl+W`: Xóa word trước
- `Ctrl+G`: Mở prompt trong external editor (rất hữu ích cho prompts dài)
- `\` + Enter: Xuống dòng nhanh
- `Ctrl+V`: Paste ảnh từ clipboard (Mac/Linux)

### C. Voice Input (giao tiếp nhanh hơn)
- Claude Code đã có built-in voice mode
- Alternative tools: superwhisper, MacWhisper, Super Voice Assistant
- Nói với Claude như nói với bạn - nó đủ thông minh để hiểu typo

### D. Cmd+A / Ctrl+A = Best friend
- Khi Claude không fetch được URL (vd Reddit), **Select All → Copy → Paste** trực tiếp vào Claude
- Áp dụng cho terminal output, YouTube transcripts, Gmail threads

---

## Workflow Best Practices

### 1. **Proactive context management**
- Tắt `auto-compact` trong `/config`
- Khi conversation dài, yêu cầu Claude tạo `HANDOFF.md`:
 > "Put the rest of the plan in HANDOFF.md. Explain what you tried, what worked, what didn't, so the next agent with fresh context can continue."
- Start new session, chỉ pass file path: `> HANDOFF.md`

### 2. **Multitasking với terminal tabs**
- Mở 3-4 Claude Code instances cùng lúc
- Mỗi tab = 1 task/branch
- **Không làm quá 4 tasks song song** (bạn sẽ lost track)

### 3. **Git workflow với Claude**
- Để Claude handle commits, branching, pulling
- **Allow pull tự động, KHÔNG allow push tự động** (push có thể phá origin)
- Dùng **draft PRs** để Claude tạo PR low-risk, bạn review trước khi merge

### 4. **Verify output đủ nhiều cách**
- Ask Claude write tests → run → fix
- Dùng visual Git client (GitHub Desktop) xem diff
- Tạo draft PR trước khi merge
- Prompt yêu thích:
 > "Double check everything, every single claim you produced, and at the end make a table of what you were able to verify."

### 5. **Research workflow**
- Claude Code là **Google/Deep Research replacement**
- Cho GitHub Actions failures: "dig into this issue, find root cause"
- Dùng `gh` CLI để Claude access GitHub info
- Dùng MCP (Playwright, Chrome DevTools) cho web research

---

## Mindset quan trọng

### **Choose the right level of abstraction**
Vibe coding không phải lúc nào cũng xấu - tùy tình huống:
- **Vibe coding** (nhìn từ xa): OK cho one-time projects, non-critical code
- **Dig deeper** (xem từng dòng): Cần cho critical code, production
- **Không binary** - bạn điều khiển mức độ zoom

### **Be braver in the unknown**
- Với Claude Code, bạn có thể tackle codebase bạn chưa từng biết
- Iteratively ask questions, explore, experiment
- Không cần là expert trước khi bắt đầu

### **Billion token rule thay vì 10,000 hour rule**
- Cách tốt nhất học Claude = **dùng Claude** nhiều
- Consume nhiều tokens = build intuition về AI

### **Automation of automation**
- Tìm tasks bạn lặp đi lặp lại → automate
- Từ manual copy-paste → CLAUDE.md → skills → scripts
- Đây là con đường để productive hơn

---

## Resources để học thêm

- **r/ClaudeAI** subreddit: Nơi chính học tips từ community
- **Ado (@adocomplete)** on X: DevRel Anthropic, đăng tips hàng ngày trong "Advent of Claude"
- **GitHub `ykdojo/claude-code-tips`**: 45 tips chi tiết (6.6k )
- **Official docs**: https://code.claude.com/docs/en/best-practices
- **`/release-notes`**: Check features mới thường xuyên

---

## ️ Quick Warnings cho Beginners

1. **KHÔNG dùng `--dangerously-skip-permissions`** trên host machine - chỉ dùng trong container
2. **Audit approved commands** thường xuyên (dùng tool `cc-safe`) - có case Claude `rm -rf ~/` xóa cả home directory
3. **KHÔNG để CLAUDE.md quá phức tạp** - keep it simple, review định kỳ
4. **Không spam nhiều tasks** - max 3-4 concurrent sessions
