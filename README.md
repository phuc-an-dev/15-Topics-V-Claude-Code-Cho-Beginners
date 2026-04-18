# 15 Topics Ve Claude Code Cho Beginners

Tai lieu nay cung cap kien thuc chuyen sau ve Claude Code CLI, mot cong cu ho tro lap trinh theo mo hinh Agentic Workflow tu Anthropic. Muc tieu cua dự án là giúp người dùng tối ưu hóa Context Window, quản lý Token Limit và xây dựng quy trình làm việc chuyên nghiệp với AI Agent.

## Danh sach cac Topic chi tiet

| STT | File | Noi dung ky thuat |
|:---|:---|:---|
| 1 | [01-best-practices.md](./01-best-practices-using-claude.md) | Cac nguyen tac cot loi ve Context Management va Problem Breakdown. |
| 2 | [02-react-typescript.md](./02-best-skills-react-typescript.md) | Skills chuyen biet cho React Ecosystem va Type Safety. |
| 3 | [03-spring-boot.md](./03-best-skills-spring-boot.md) | Skills cho Java Spring Boot, Dependency Injection va Unit Testing. |
| 4 | [04-developer-skills.md](./04-best-skills-developer.md) | Ky nang Software Engineering can thiet khi lam viec voi AI. |
| 5 | [05-token-limit.md](./05-khong-bi-token-limit.md) | Chien luoc toi uu hoa Token Usage va tranh Rate Limit. |
| 6 | [06-config-claude.md](./06-best-practice-cau-hinh-claude.md) | Cau hinh he thong qua settings.json va CLAUDE.md. |
| 7 | [07-concepts.md](./07-best-concepts.md) | Cac khai niem nang cao nhu TDD, Refactoring va Design Patterns. |
| 8 | [08-github-integration.md](./08-claude-with-github.md) | Tu dong hoa Pull Request, Code Review va Git Workflow. |
| 9 | [09-obsidian-integration.md](./09-claude-with-obsidian.md) | Quan ly kien thuc va Second Brain thong qua Obsidian MCP. |
| 10 | [10-agent-best-practice.md](./10-best-practice-using-agent.md) | Quy trinh dieu phoi Agent va Subagents cho cac task phuc tap. |
| 11 | [11-manage-memory.md](./11-manage-claude-memory.md) | Quan ly Session Memory va luu tru trang thai dai han. |
| 12 | [12-session-preservation.md](./12-session-memory-preservation.md) | Ky thuat bao ton Context giua cac phien lam viec khac nhau. |
| 13 | [13-best-hooks-for-developer.md](./13-best-hooks-for-developer.md) | Thiet lap Lifecycle Hooks de tu dong hoa verify code. |
| 14 | [14-plan-mode.md](./14-plan-mode-best-practice.md) | Su dung Plan Mode de thiet ke kien truc truoc khi Implementation. |
| 15 | [15-mistakes.md](./15-mistakes-anti-patterns.md) | Danh sach Anti-patterns va cac sai lam pho bien can tranh. |

## Cac nguyen tac ky thuat cot loi

### 1. Context Management
Context Window la tai nguyen quan trong nhat. Khi Context day, hieu suat cua AI se suy giam (Performance Degradation). Nguoi dung can thuc hien:
- Su dung `/clear` de xoa bo Context cu khi bat dau Task moi.
- Dung `/compact` de nen Context hien tai, giu lai nhung thong tin quan trong nhat.
- Tao `HANDOFF.md` de chuyen tiep trang thai du an giua cac Session.

### 2. Agentic Workflow (Explore - Plan - Execute - Verify)
Thay vi de AI code ngay lap tuc, hay tuan thu quy trinh:
- Explore: AI doc file va hieu Codebase.
- Plan: AI de xuat Implementation Plan chi tiet trong Plan Mode.
- Execute: Thuc thi code theo tung buoc nho.
- Verify: Chay Unit Tests, Linter va Type Check de dam bao chat luong.

### 3. TDD (Test Driven Development) voi AI
Claude Code hoat dong hieu qua nhat khi co muc tieu ro rang. Hay yeu cau AI viet Failing Test truoc, sau do moi viet Implementation de pass test do. Day la cach tot nhat de kiem soat tinh chinh xac cua AI.

### 4. Subagents va Delegation
Doi voi cac Task lon, hay su dung Subagents de thuc hien cac cong viec con nhu Research hoac Document Analysis. Viec nay giup giu cho Main Context luon sach se va tap trung vao logic chinh.

## Thiet lap he thong (System Configuration)

### Security va Permissions
Cau hinh file `~/.claude/settings.json` de bao ve du lieu nhay cam:
- Deny permissions doi voi file `.env` va thu muc `secrets`.
- Han che quyen Read/Write o cac thu muc he thong.
- Khong su dung flag `--dangerously-skip-permissions` tren Host Machine.

### CLAUDE.md Guideline
File `CLAUDE.md` nen duoc coi la mot tai lieu huong dan convention cho AI:
- Giu file duoi 200 dong de tranh AI bo qua thong tin.
- Chi ghi nhung quy tac bat buoc (Coding Style, Test Commands).
- Cap nhat dinh ky de loai bo cac quy tac khong con phu hop.

## Slash Commands va Shortcuts quan trong

- `/clear`: Bat dau Session moi.
- `/compact`: Nen Context.
- `/usage`: Kiem tra Token Usage va Rate Limit.
- `/plan`: Chuyen sang Plan Mode.
- `Ctrl + G`: Mo Prompt trong External Editor de viet yeu cau phuc tap.
- `Esc + Esc`: Rewind ve checkpoint truoc do.

## Anti-patterns can tranh

- Kitchen Sink Session: Lam qua nhieu task khong lien quan trong cung mot session.
- Trust without Verification: Chap nhan code cua AI ma khong chay Test hoac Review.
- Over-specified CLAUDE.md: Dua qua nhieu chi tiet vao file cau hinh khien AI bi nhiễu thong tin.
- Correcting Loops: Tiep tuc sua loi cua AI qua 2 lan ma khong `/clear` de lam moi Context.

---
Tai lieu duoc tong hop tu Anthropic Official Documentation, r/ClaudeCode va cac chuyen gia AI Engineering.
