# Topic 15: 15 Mistakes & Anti-Patterns Cần Tránh Khi Work Với Claude

> **Nguồn**: Anthropic official best practices docs, Reddit r/ClaudeAI, Medium articles (dev.to, 0xhagen), `ykdojo/claude-code-tips`

---

## Top 15 Mistakes (xếp theo mức độ phổ biến & nghiêm trọng)

### 1. **The Over-Specified CLAUDE.md** (rất phổ biến)
**Triệu chứng**: CLAUDE.md quá dài, Claude ignore rules quan trọng.
> Quote từ Anthropic: "If your CLAUDE.md is too long, Claude **ignores half of it** because important rules get lost in the noise."

**Fix**:
- Ruthlessly prune
- Với mỗi dòng, hỏi: "Xóa dòng này có làm Claude sai không?" Nếu không → xóa
- Chuyển rules lặp lại thành **hooks** (deterministic)

---

### 2. **The Kitchen Sink Session** 
**Triệu chứng**: Bắt đầu 1 task → nhảy sang task khác → quay lại task đầu → context đầy rác.

**Fix**: Dùng `/clear` giữa các tasks không liên quan.

---

### 3. **Correcting Over and Over** 
**Triệu chứng**: Claude làm sai, bạn sửa, vẫn sai, sửa tiếp... → context đầy "failed approaches".

**Fix (Anthropic official)**: Sau **2 lần correction thất bại**, `/clear` và viết lại prompt tốt hơn dựa trên những gì đã học.

> "A clean session with a better prompt **almost always outperforms** a long session with accumulated corrections."

---

### 4. **The Trust-Then-Verify Gap** 
**Triệu chứng**: Claude produce code "nhìn ổn" nhưng không handle edge cases.

**Fix**:
- **Luôn** provide verification (tests, scripts, screenshots)
- **"If you can't verify it, don't ship it"** — Anthropic
- Prompt yêu thích: "Double check every claim, make a verification table at the end"

---

### 5. **The Infinite Exploration** 
**Triệu chứng**: "Investigate X" (no scope) → Claude đọc hàng trăm files → context đầy.

**Fix**:
- Scope investigations hẹp: "Check auth flow in `src/auth/`, especially token refresh"
- Dùng **subagents** để research (có context riêng, không pollute main)

---

### 6. **Không dùng Plan Mode khi nên dùng** 
**Triệu chứng**: Claude jump straight to code → solve wrong problem.

> Case study (dev.to): "With Claude Code in VS Code, if you make the mistake of describing the next ticket to lay some groundwork, it will happily trot off and start making code changes **without any discussion at all**."

**Fix**: Task phức tạp/multi-file → **Plan Mode** (Shift+Tab). Task đơn giản (fix typo) → skip planning.

---

### 7. **Vague Prompts (không đủ context)** 
**Triệu chứng**: "fix the login bug" → Claude đoán mò → sai hướng.

**Fix** (before/after):

| Before | After |
|----------|---------|
| "fix the login bug" | "users report login fails after session timeout. Check auth flow in `src/auth/`, especially token refresh. Write a failing test that reproduces, then fix." |
| "add tests for foo.py" | "write test for foo.py covering the edge case where user is logged out. avoid mocks." |
| "make dashboard look better" | "[paste screenshot] implement this design. take screenshot of result, compare, list differences, fix." |

---

### 8. **Dùng `--dangerously-skip-permissions` trên Host Machine** ️ CỰC NGUY HIỂM
**Case thật trên Reddit**: Claude ran `rm -rf tests/ patches/ plan/ ~/` → **xóa sạch home directory**.

**Fix**:
- KHÔNG BAO GIỜ dùng flag này trên host
- Chỉ dùng trong **container** (SafeClaw, Docker)
- Audit approved commands định kỳ với `npx cc-safe .`

---

### 9. **Không Review Code Claude Viết Ra** 
**Triệu chứng**: Vibe coding 100%, không hiểu code mình có.

**Anti-pattern**:
> "AI confused the name of the python file with the API URL, put it into API documentation, leading to later code change. AI will make changes **without checking or raising them with you**." — dev.to

**Fix**:
- Ask questions: "Why did you make this change?" "Why add this line?"
- Review với GitHub Desktop / visual diff tool
- Tạo draft PR, review carefully trước khi merge

---

### 10. **Claude Overcomplicated Code** ️
**Triệu chứng**: Claude có bias viết quá nhiều code, thêm changes không ai ask.

**Fix**:
- Ask: "Simplify this" hoặc "why did you add this?"
- Áp dụng cho cả prose (Claude hay repeat idea ở paragraph cuối)

---

### 11. **Monolithic Codebase Không Refactor** 
**Triệu chứng**: Claude struggle với codebase lớn → context cạn nhanh → regression sau compaction.

> Từ 0xhagen (Medium): "After compaction, Claude **forgets recent context** and reverts to earlier understanding of your project."

**Fix** (từ feedback này là signal để refactor):
- **Module Extraction**: tách monolith thành services
- **Feature Slicing**: organize by feature (vertical), không phải layer (horizontal)
- Đặt `.claudeignore` ở mỗi package

---

### 12. **Không Dùng Subagents Cho Research** 
**Triệu chứng**: Bạn bảo Claude research → nó đọc 50 files → main context đầy → implementation chập chờn.

**Fix**:
> "Since context is your fundamental constraint, subagents are **one of the most powerful tools available**." — Anthropic

Prompt: "Use subagents to investigate how auth handles token refresh."

---

### 13. **Không Disable Auto-Compact Nếu Có Patches** 
**Triệu chứng**:
- Bạn patch system prompt để tiết kiệm tokens
- Claude Code auto-update → phá hết patches
- Hoặc auto-compact trước khi hook kịp fire

**Fix**:
```json
{
 "env": {
  "DISABLE_AUTOUPDATER": "1"
 }
}
```
Và `/config` → Auto-compact → false.

---

### 14. **Không Deny `.env` và Secrets** 
**Triệu chứng**: Claude đọc `.env` vào context → leak API keys trong conversation logs/debug output.

**Fix**:
```json
{
 "permissions": {
  "deny": [
   "Read(./.env)",
   "Read(./.env.*)",
   "Read(./secrets/**)",
   "Read(./config/credentials.json)"
  ]
 }
}
```
️ Cảnh báo: Reddit reports cho thấy deny rules **đôi khi bị bỏ qua** → kết hợp với `.gitignore` và không store secrets trong project folder.

---

### 15. **Quá Nhiều Parallel Sessions** 
**Triệu chứng**: Mở 10 Claude Code tabs → lost track → merge conflicts → confusion.

**Fix** (ykdojo rule):
- **Max 3-4 concurrent sessions**
- Dùng "cascade method": task mới → tab phải, sweep left-to-right
- Tab trái nhất cho process persistent (vd voice transcription)
- Dùng **git worktrees** cho parallel branch work để tránh conflict

---

## Bonus Anti-Patterns (thường gặp)

### 16. **Không Biết Dùng `Esc`**
- `Esc`: Stop Claude giữa chừng, context giữ nguyên
- `Esc + Esc` / `/rewind`: Rewind menu, restore state
- Không dùng → phải chờ Claude finish, waste tokens

### 17. **Quên `/release-notes`**
- Features mới hàng tuần
- Không update → miss out `/fork`, auto mode, tool search, etc.

### 18. **Đổ Mọi Thứ vào CLAUDE.md thay vì Skills**
- CLAUDE.md = load **mọi conversation** (waste tokens)
- Skills = load **khi cần** (Claude tự invoke, hoặc `/skill-name`)
- Rule: Domain-specific/occasional → Skills. Broad/always → CLAUDE.md.

### 19. **Không Có Handoff Document Khi Session Quá Dài**
- Khi context 85%+, không tạo `HANDOFF.md` trước khi `/clear`
- → Mất toàn bộ progress
- Fix: Ask Claude write HANDOFF.md với "what tried, what worked, what didn't"

### 20. **Agentic Systems Primed to Act, Not Interact**
> "Agentic systems are primed to **act, not interact**, which is a tragedy because the one thing LLMs were really good at was verbal exploration of ideas." — dev.to

**Fix**:
- Dùng Claude để **discuss** trước khi code
- Prompt: "Interview me using AskUserQuestion tool. Ask about technical implementation, UI/UX, edge cases."
- Tạo SPEC.md trước khi start fresh session implementation

---

## Tổng Kết Nhanh - Do & Don't

| DON'T | DO |
|---------|------|
| CLAUDE.md dài dòng | Ngắn gọn, prune thường xuyên |
| Long session đủ mọi topic | `/clear` giữa tasks |
| Correct 5 lần cùng 1 bug | Sau 2 lần fail → `/clear` + prompt mới |
| Trust Claude blindly | Verify với tests, screenshots, PR review |
| "Investigate X" (vague) | Scope hẹp + dùng subagents |
| Skip Plan Mode | Plan Mode cho multi-file changes |
| Vague prompts | File references, constraints, examples |
| `--dangerously-skip-permissions` trên host | Chỉ dùng trong container |
| Không review Claude's code | Ask "why?" questions, visual diff |
| Dùng `.env` không deny | `permissions.deny` + `.gitignore` |
| Mở 10 sessions | Max 3-4 với git worktrees |
| Đổ mọi thứ vào CLAUDE.md | Skills cho domain-specific |
| Let Claude act immediately | Let Claude interview you first |

---

## Mindset Shift

### **Từ cũ → Sang mới**:
- Don't rely on **finishing in one pass**
- Don't let AI make **judgment calls**
- Make tasks **interruptible + resumable + directionally locked**
- **You** handle judgment, **AI** handles pattern matching
- When structure holds, crash is just a **local event** (không phải thảm họa)

---

## Sources
- Anthropic Official: https://code.claude.com/docs/en/best-practices (section: "Avoid common failure patterns")
- dev.to "Pitfalls of Claude Code": https://dev.to/cheetah100/pitfalls-of-claude-code-1nb6
- Medium "Why Your Claude Code Sessions Keep Failing": https://0xhagen.medium.com/why-your-claude-code-sessions-keep-failing-and-how-to-fix-it-62d5a4229eaf
- Medium "What to Do When Claude Code Crashes": https://medium.com/@zia8he/what-to-do-when-claude-code-crashes-6ee48c3f2ca5
- `ykdojo/claude-code-tips` - Tip 33 (audit commands), Tip 40 (simplify)
- Reddit r/ClaudeAI: Case study "rm -rf home directory"
