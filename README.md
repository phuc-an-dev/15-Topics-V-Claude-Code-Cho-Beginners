# 🚀 15 Topics Về Claude Code Cho Beginners (Tiếng Việt)

Chào mừng bạn đến với bộ tài liệu tổng hợp kiến thức chuyên sâu về **Claude Code CLI** - công cụ hỗ trợ lập trình agentic mạnh mẽ từ Anthropic. Tài liệu này được biên soạn nhằm giúp các lập trình viên mới bắt đầu (beginners) làm chủ Claude Code một cách hiệu quả, tiết kiệm chi phí (tokens) và tối ưu hóa quy trình làm việc.

---

## 🗂️ Cấu Trúc Dự Án

Dự án bao gồm 15 chủ đề quan trọng, đi từ những kiến thức cơ bản nhất đến các kỹ năng nâng cao:

| File | Nội dung chính |
|:-----|:---------------|
| [00-SUMMARY.md](./00-SUMMARY.md) | 📚 Tổng hợp tóm tắt toàn bộ 15 topics. |
| [01-best-practices.md](./01-best-practices-using-claude.md) | 🎯 Các nguyên tắc cốt lõi khi sử dụng Claude. |
| [02-react-typescript.md](./02-best-skills-react-typescript.md) | ⚛️ Skills chuyên biệt cho React & TypeScript. |
| [03-spring-boot.md](./03-best-skills-spring-boot.md) | 🍃 Skills chuyên biệt cho Spring Boot. |
| [04-developer-skills.md](./04-best-skills-developer.md) | 🛠️ Các kỹ năng chung cho lập trình viên. |
| [05-token-limit.md](./05-khong-bi-token-limit.md) | 📉 Cách tối ưu và tránh giới hạn token. |
| [06-config-claude.md](./06-best-practice-cau-hinh-claude.md) | ⚙️ Hướng dẫn cấu hình `.claude` và `settings.json`. |
| [07-concepts.md](./07-best-concepts.md) | 🧠 Các khái niệm quan trọng (TDD, Agentic Workflow). |
| [08-github-integration.md](./08-claude-with-github.md) | 🐙 Kết hợp Claude với GitHub (CLI, PRs). |
| [09-obsidian-integration.md](./09-claude-with-obsidian.md) | 📓 Quản lý kiến thức với Obsidian MCP. |
| [10-agent-best-practice.md](./10-best-practice-using-agent.md) | 🤖 Cách sử dụng Agent và Subagent hiệu quả. |
| [11-manage-memory.md](./11-manage-claude-memory.md) | 💾 Quản lý bộ nhớ của Claude giữa các phiên làm việc. |
| [12-session-preservation.md](./12-session-memory-preservation.md) | 🔄 Giữ ngữ cảnh (context) bền vững. |
| [13-hooks-for-dev.md](./13-best-hooks-for-developer.md) | 🪝 Sử dụng Hooks để tự động hóa quy trình. |
| [14-plan-mode.md](./14-plan-mode-best-practice.md) | 📋 Cách sử dụng Plan Mode chuyên nghiệp. |
| [15-mistakes.md](./15-mistakes-anti-patterns.md) | 🚨 15 lỗi thường gặp và cách khắc phục. |

---

## 🌟 5 Key Takeaways (Đọc Ngay!)

1.  **Context là "sữa tươi":** Context hoạt động tốt nhất khi ngắn gọn và mới. Đừng ngại dùng `/clear` thường xuyên.
2.  **Plan trước khi Code:** Luôn dùng `Shift + Tab` (Plan Mode) để thiết kế giải pháp trước khi để Claude bắt đầu sửa file.
3.  **Quy tắc 2 lần sửa:** Nếu Claude sai đến lần thứ 2 cho cùng một lỗi, hãy `/clear` và cung cấp một prompt tốt hơn hoặc cập nhật `CLAUDE.md`.
4.  **Kiểm chứng là bắt buộc:** "Nếu bạn không thể verify, đừng ship". Luôn yêu cầu Claude viết test hoặc tự mình kiểm tra output.
5.  **Tự động hóa sự tự động:** Nếu một tác vụ lặp lại >3 lần, hãy biến nó thành một **Skill** hoặc **Hook**.

---

## 🛠️ Cài đặt cơ bản (Quick Start)

### 1. Cấu hình bảo mật (`~/.claude/settings.json`)
Luôn chặn Claude đọc các file nhạy cảm:
```json
{
  "permissions": {
    "deny": ["Read(./.env)", "Read(./.env.*)", "Read(./secrets/**)"]
  }
}
```

### 2. Các lệnh Slash (/) cần nhớ
- `/clear`: Xóa session cũ, làm sạch context (Quan trọng nhất!).
- `/compact`: Nén context để tiết kiệm token.
- `/usage`: Kiểm tra giới hạn sử dụng hiện tại.
- `/plan`: Chuyển sang chế độ lập kế hoạch.

---

## 🚨 Cảnh báo quan trọng
- **KHÔNG** dùng flag `--dangerously-skip-permissions` trên máy thật (host), chỉ dùng trong môi trường container an toàn.
- **KHÔNG** để file `CLAUDE.md` quá dài (>200 dòng), vì Claude sẽ bắt đầu bỏ qua các quy tắc quan trọng.

---

## 📈 Lộ trình học tập (Learning Path)
- **Tuần 1:** Làm quen với các lệnh cơ bản và thiết lập môi trường.
- **Tuần 2:** Học cách quản lý context và sử dụng Plan Mode.
- **Tuần 3:** Cài đặt các Skills từ cộng đồng (như `obra/superpowers`).
- **Tuần 4:** Tự tạo Hooks và tối ưu hóa workflow cá nhân.

---

*Tài liệu này được tổng hợp từ các nguồn uy tín: Reddit r/ClaudeCode, Anthropic Official Docs, và các repo GitHub hàng đầu.*

**Chúc bạn có một hành trình lập trình tuyệt vời cùng Claude Code! 🚀**
