# 15 Chủ đề về Claude Code dành cho người mới bắt đầu

Tài liệu này cung cấp kiến thức chuyên sâu về Claude Code CLI, một công cụ hỗ trợ lập trình theo mô hình Agentic Workflow từ Anthropic. Mục tiêu của dự án là giúp người dùng tối ưu hóa Context Window, quản lý Token Limit và xây dựng quy trình làm việc chuyên nghiệp với AI Agent.

## Danh sách các Topic chi tiết

| STT | File | Nội dung kỹ thuật |
|:---|:---|:---|
| 1 | [01-best-practices.md](./01-best-practices-using-claude.md) | Các nguyên tắc cốt lõi về Context Management và Problem Breakdown. |
| 2 | [02-react-typescript.md](./02-best-skills-react-typescript.md) | Skills chuyên biệt cho React Ecosystem và Type Safety. |
| 3 | [03-spring-boot.md](./03-best-skills-spring-boot.md) | Skills cho Java Spring Boot, Dependency Injection và Unit Testing. |
| 4 | [04-developer-skills.md](./04-best-skills-developer.md) | Kỹ năng Software Engineering cần thiết khi làm việc với AI. |
| 5 | [05-token-limit.md](./05-khong-bi-token-limit.md) | Chiến lược tối ưu hóa Token Usage và tránh Rate Limit. |
| 6 | [06-config-claude.md](./06-best-practice-cau-hinh-claude.md) | Cấu hình hệ thống qua settings.json và CLAUDE.md. |
| 7 | [07-concepts.md](./07-best-concepts.md) | Các khái niệm nâng cao như TDD, Refactoring và Design Patterns. |
| 8 | [08-github-integration.md](./08-claude-with-github.md) | Tự động hóa Pull Request, Code Review và Git Workflow. |
| 9 | [09-obsidian-integration.md](./09-claude-with-obsidian.md) | Quản lý kiến thức và Second Brain thông qua Obsidian MCP. |
| 10 | [10-agent-best-practice.md](./10-best-practice-using-agent.md) | Quy trình điều phối Agent và Subagents cho các tác vụ phức tạp. |
| 11 | [11-manage-memory.md](./11-manage-claude-memory.md) | Quản lý Session Memory và lưu trữ trạng thái dài hạn. |
| 12 | [12-session-preservation.md](./12-session-memory-preservation.md) | Kỹ thuật bảo tồn Context giữa các phiên làm việc khác nhau. |
| 13 | [13-best-hooks-for-developer.md](./13-best-hooks-for-developer.md) | Thiết lập Lifecycle Hooks để tự động hóa việc verify code. |
| 14 | [14-plan-mode.md](./14-plan-mode-best-practice.md) | Sử dụng Plan Mode để thiết kế kiến trúc trước khi Implementation. |
| 15 | [15-mistakes.md](./15-mistakes-anti-patterns.md) | Danh sách Anti-patterns và các sai lầm phổ biến cần tránh. |

## Các nguyên tắc kỹ thuật cốt lõi

### 1. Context Management
Context Window là tài nguyên quan trọng nhất. Khi Context đầy, hiệu suất của AI sẽ suy giảm (Performance Degradation). Người dùng cần thực hiện:
- Sử dụng lệnh `/clear` để xóa bỏ Context cũ khi bắt đầu Task mới.
- Dùng lệnh `/compact` để nén Context hiện tại, giữ lại những thông tin quan trọng nhất.
- Tạo file `HANDOFF.md` để chuyển tiếp trạng thái dự án giữa các Session.

### 2. Agentic Workflow (Explore - Plan - Execute - Verify)
Thay vì để AI thực hiện viết code ngay lập tức, hãy tuân thủ quy trình:
- Explore: AI đọc file và tìm hiểu Codebase.
- Plan: AI đề xuất Implementation Plan chi tiết trong Plan Mode.
- Execute: Thực thi viết code theo từng bước nhỏ.
- Verify: Chạy Unit Tests, Linter và Type Check để đảm bảo chất lượng.

### 3. TDD (Test Driven Development) với AI
Claude Code hoạt động hiệu quả nhất khi có mục tiêu rõ ràng. Hãy yêu cầu AI viết Failing Test trước, sau đó mới viết Implementation để vượt qua (pass) bài kiểm tra đó. Đây là cách tốt nhất để kiểm soát tính chính xác của AI.

### 4. Subagents và Delegation
Đối với các Task lớn, hãy sử dụng Subagents để thực hiện các công việc con như Research hoặc Document Analysis. Việc này giúp giữ cho Main Context luôn sạch sẽ và tập trung vào logic chính.

## Thiết lập hệ thống (System Configuration)

### Security và Permissions
Cấu hình file `~/.claude/settings.json` để bảo vệ dữ liệu nhạy cảm:
- Deny permissions đối với file `.env` và thư mục `secrets`.
- Hạn chế quyền Read/Write ở các thư mục hệ thống.
- Tuyệt đối không sử dụng flag `--dangerously-skip-permissions` trên Host Machine.

### CLAUDE.md Guideline
File `CLAUDE.md` nên được coi là một tài liệu hướng dẫn quy chuẩn (convention) cho AI:
- Giữ file dưới 200 dòng để tránh AI bỏ qua thông tin.
- Chỉ ghi những quy tắc bắt buộc như Coding Style hoặc Test Commands.
- Cập nhật định kỳ để loại bỏ các quy tắc không còn phù hợp.

## Slash Commands và Shortcuts quan trọng

- `/clear`: Bắt đầu Session mới.
- `/compact`: Nén Context.
- `/usage`: Kiểm tra Token Usage và Rate Limit.
- `/plan`: Chuyển sang Plan Mode.
- `Ctrl + G`: Mở Prompt trong External Editor để viết yêu cầu phức tạp.
- `Esc + Esc`: Quay lại (Rewind) về checkpoint trước đó.

## Anti-patterns cần tránh

- Kitchen Sink Session: Làm quá nhiều tác vụ không liên quan trong cùng một phiên làm việc.
- Trust without Verification: Chấp nhận code của AI mà không chạy thử nghiệm (Test) hoặc Review.
- Over-specified CLAUDE.md: Đưa quá nhiều chi tiết vào file cấu hình khiến AI bị nhiễu thông tin.
- Correcting Loops: Tiếp tục sửa lỗi của AI quá 2 lần mà không dùng `/clear` để làm mới Context.

---
Tài liệu được tổng hợp từ Anthropic Official Documentation, r/ClaudeCode và các chuyên gia AI Engineering.
