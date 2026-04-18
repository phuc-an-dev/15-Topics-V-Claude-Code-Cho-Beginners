# Topic 5: Best Way Để Không Bị Token Limit

> **Nguồn**: Anthropic Official docs (code.claude.com, support.claude.com), Reddit r/ClaudeAI, `ykdojo/claude-code-tips`, MindStudio blog

---

## 🧠 Hiểu Về Token Limit Trước

### Hai layers limit (quan trọng!)
1. **Context window** (per-conversation): 200K tokens (Pro/Max), 500K (Enterprise một số models)
2. **Rate limits** (per-5-hours và per-week): quota tổng của account

> Key insight: "Almost every 'I burned through my limit by lunchtime' report traces back to **very long sessions that were never cleared**." — Anthropic support

### Token consumption theo thứ tự phổ biến
| Nguồn tốn tokens | % điển hình |
|-----------------|-------------|
| System prompt + tool definitions | ~10% (19K tokens) ngay từ start |
| File reads (tool outputs) | Lớn nhất - mỗi file có thể vài K tokens |
| MCP server tool definitions | 5-10K+ nếu nhiều servers |
| CLAUDE.md (load mỗi turn) | Scale theo độ dài |
| Conversation history | Accumulate theo turns |
| Thinking blocks | Bị strip tự động giữa turns |

---

## 🎯 15 Chiến Thuật Tiết Kiệm Tokens (theo priority)

### ⭐ Priority 1: Session Management

#### 1. **`/clear` khi chuyển topic**
- **Cú click đơn quan trọng nhất** - single most effective lever
- Dùng khi: start task mới, corrected Claude 2+ lần, topic switch
- Giữ lại: CLAUDE.md, project files, settings

#### 2. **`/compact <instructions>` khi cần continue**
- Dùng khi đang mid-task nhưng context đầy
- Có thể guide: `/compact Focus on the API changes and file paths`
- Customize trong CLAUDE.md: `"When compacting, always preserve modified files list and test commands"`

#### 3. **Quy tắc vàng**: `/clear` cho task mới, `/compact` để continue task hiện tại

#### 4. **`Esc + Esc` hoặc `/rewind` để compact từng phần**
- Select checkpoint cụ thể, compact từ đó
- Giữ earlier context quan trọng, condense phần giữa

#### 5. **`/btw` cho side questions**
- Câu hỏi nhanh không cần stay trong context
- Answer hiện ở overlay, không vào conversation history

---

### ⭐ Priority 2: Reduce System Overhead

#### 6. **Enable Tool Search (lazy-load MCP tools)**
```json
{ "env": { "ENABLE_TOOL_SEARCH": "true" } }
```
- Tiết kiệm ~5-10K tokens nếu có nhiều MCP servers
- Từ v2.1.7, auto enable khi MCP tools > 10% context
- MCP tools chỉ load khi Claude cần

#### 7. **Slim Down System Prompt (advanced)**
- System prompt default ~19K tokens
- Patch có thể cut xuống ~9K (tiết kiệm 50% overhead)
- **Trade-off**: Ít "safety rails" hơn, nhưng context rộng hơn
- Repo: `ykdojo/claude-code-tips` Tip 15 có patch scripts
- ⚠️ Phải `DISABLE_AUTOUPDATER=1` nếu không update sẽ phá patches

#### 8. **`--system-prompt-file` flag (alternative chính thức)**
Thay thế system prompt bằng custom file ngắn gọn hơn.

---

### ⭐ Priority 3: CLAUDE.md Discipline

#### 9. **Keep CLAUDE.md siêu ngắn**
> "The CLAUDE.md file is read at the **start of every session**, so every word costs tokens **every time**." — MindStudio

- Cut anything descriptive/aspirational
- Rule: "If it doesn't change how Claude behaves, delete it"
- Target: < 50-100 lines

#### 10. **Dùng Skills thay vì CLAUDE.md cho domain-specific**
- CLAUDE.md = load mọi conversation (luôn tốn tokens)
- Skills = lazy-load khi Claude cần (tiết kiệm 100% khi không dùng)
- Move workflows occasional → `.claude/skills/`

---

### ⭐ Priority 4: Smart File Handling

#### 11. **Dùng `.claudeignore` như `.gitignore`**
- Chặn Claude đọc: `node_modules`, `dist/`, `build/`, `*.lock`, `*.log`, datasets lớn
- Đặt ở project root và mỗi package (monorepo)

#### 12. **Prompt precision - reference bare path, không `@`**
| ❌ Wastes tokens | ✅ Efficient |
|------------------|--------------|
| `@auth.ts fix the token bug` (inject toàn bộ file + CLAUDE.md tree) | "look at `validateToken` function in `src/auth.ts`" (Claude mở selective) |
| Paste cả stack trace 200 lines | Trim về 20-30 lines relevant |
| Paste data dump | Lưu file trên disk, reference path |

#### 13. **Specific prompts beat broad scanning**
| ❌ Broad (tốn context) | ✅ Narrow (tiết kiệm) |
|-----------------------|----------------------|
| "improve this codebase" | "add input validation to login function in `auth.ts`" |
| "fix the bugs" | "fix the null check bug in `useUser.ts` line 42" |
| "investigate X" | "use subagents to investigate X" |

---

### ⭐ Priority 5: Use Subagents

#### 14. **Subagents = Separate Context Windows**
> "Since context is your fundamental constraint, subagents are **one of the most powerful tools available**." — Anthropic

- Subagents explore trong context riêng
- Report back **summary** vào main context
- Dùng cho: codebase investigation, security review, edge case checking

**Prompt examples**:
```
Use subagents to investigate how our auth handles token refresh.
```
```
Use a subagent to review this code for edge cases.
```

---

### ⭐ Priority 6: Task Structuring

#### 15. **Break Down Large Tasks**
- Nếu regularly hit compaction → answer không phải bigger context → **smaller tasks**
- Pattern Writer/Reviewer trong parallel sessions (mỗi session fresh context)

---

## 💡 Pro Patterns

### Pattern 1: **HANDOFF.md Technique**
Khi sắp đầy context, ask Claude write `HANDOFF.md`:

```
Put the rest of the plan in HANDOFF.md. Explain what you've tried,
what worked, what didn't work, so the next agent with fresh context
can load that file and nothing else to finish this task.
```

Benefits:
- Deterministic (không phải lossy summary)
- Persist across sessions
- Review được trước khi `/clear`

### Pattern 2: **Model Switching**
- `/model sonnet` cho most coding work (fast, efficient)
- `/model opus` cho hard reasoning, cross-cutting refactors
- `/model haiku` cho simple tasks (cực kỳ verbose-efficient)
- `/model opus-plan` - plan với Opus, execute với Sonnet (save big)

### Pattern 3: **Construct Prompts in Notepad**
> "Write out what you want in a separate notepad, organize it, then paste whole thing." — limitededitionjonathan blog

- Batch thoughts → 1 message thay vì 5 half-formed
- Force thinking trước khi spend tokens

### Pattern 4: **Fork/Half-clone Conversations**
- `/fork` hoặc `--fork-session` để branch off
- `/half-clone` (từ dx plugin) keep only later half, cut 50% tokens

### Pattern 5: **Projects + RAG (Claude.ai web/app)**
- Uploads vào Project Knowledge thay vì paste vào chat
- RAG only loads relevant chunks (up to 10x normal context)
- Project Instructions = persistent, lightweight context

---

## 🚨 Token Drain Culprits (tránh!)

### 1. **Kitchen Sink Sessions**
Multiple unrelated tasks → context đầy rác → performance drop.

### 2. **Agent Teams Overuse**
> "Agent teams use ~7x more tokens than standard sessions." — Anthropic

- Mỗi teammate có context window riêng
- Chỉ dùng cho tasks parallel rõ ràng
- Clean up khi xong: idle teammates vẫn tốn tokens

### 3. **Verbose Output Mode**
- `--verbose` hữu ích debug, tốn tokens prod
- Tắt `--verbose` khi run production workflows

### 4. **Broad "Investigate" Prompts**
"Investigate the codebase" → Claude đọc 100+ files → context explode.

### 5. **Không có Verification Loop**
Claude write code wrong → bạn correct 5 lần → context đầy failed attempts.

---

## 📊 Monitoring Token Usage

### Tools để track
| Tool | Mục đích |
|------|----------|
| `/context` | Xem context breakdown (system, tools, memory, conversation) |
| `/usage` | Rate limits còn lại (5h + weekly) |
| `/cost` | Running spend (API key users) |
| `/stats` | GitHub-style activity graph |
| Custom status line | Real-time token % ở bottom |

### Status line template (ykdojo)
```
Opus 4.5 | 📁my-project | 🔀main | ██░░░░░░░░ 18% of 200k tokens
```
→ Biết ngay khi đến 80% → plan `/clear` hoặc HANDOFF.

---

## 🎯 Action Checklist cho Beginners

### Setup một lần (tiết kiệm tokens mọi session)
- [ ] Enable `ENABLE_TOOL_SEARCH: "true"` trong settings
- [ ] Setup custom status line với token %
- [ ] Tạo `.claudeignore` với `node_modules`, `dist/`, `.next/`, etc.
- [ ] Keep CLAUDE.md < 100 lines, prune định kỳ
- [ ] Default model = Sonnet (switch Opus khi cần)

### Thói quen mỗi session
- [ ] `/clear` khi switch task (không merge tasks!)
- [ ] `/compact` khi đang giữa task mà gần hết context
- [ ] Prompt precision: reference files cụ thể, không broad scans
- [ ] Use subagents cho investigation
- [ ] Ctrl+G mở prompt trong editor (cho prompts dài)

### Khi thấy "Context Window Full"
- [ ] Đã > 80%? → Tạo HANDOFF.md → `/clear` → restart với HANDOFF
- [ ] Mid-task? → `/compact Focus on X`
- [ ] Quá nhiều failed attempts? → `/clear` + rewrite prompt better

### Khi hit Weekly Rate Limit
- [ ] `/model sonnet` hoặc `/model haiku` (ít tốn quota hơn)
- [ ] Switch sang API key tạm thời (nếu org allow)
- [ ] Review `/stats` xem pattern nào waste nhiều

---

## 📚 Sources
- Anthropic Official: https://code.claude.com/docs/en/costs
- Support article: https://support.claude.com/en/articles/14552983-models-usage-and-limits-in-claude-code
- Context windows docs: https://platform.claude.com/docs/en/build-with-claude/context-windows
- MindStudio 18 hacks: https://www.mindstudio.ai/blog/claude-code-token-management-hacks-3
- "15 Habits" article: https://limitededitionjonathan.substack.com/p/why-you-keep-hitting-claudes-usage
- `ykdojo/claude-code-tips` Tip 5, 8, 15, 23 (fork/half-clone)
- Context management: https://claudefa.st/blog/guide/mechanics/context-management
