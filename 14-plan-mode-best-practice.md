# Topic 14: Best Practices Using Claude Code with Plan Mode

> **Nguồn**: Anthropic Official docs (code.claude.com), Reddit r/ClaudeAI, `ykdojo/claude-code-tips` Tip 8

---

## 🎯 Plan Mode là gì?

**Plan Mode** = Claude chỉ **đọc và phân tích** codebase, **không** sửa file nào.
- Chỉ dùng read-only operations
- Perfect để: explore codebase mới, plan thay đổi phức tạp, review code an toàn
- Output: Implementation plan chi tiết bạn review trước khi execute

### Cách vào Plan Mode
| Cách | Command |
|------|---------|
| In-session | `Shift+Tab` (cycle: Normal → Auto-Accept → Plan) |
| Start new session | `claude --permission-mode plan` |
| Headless query | `claude --permission-mode plan -p "prompt"` |

Bạn sẽ thấy `⏸ plan mode on` ở bottom terminal.

---

## 📋 Workflow 4 Phases (Official Anthropic)

### **Phase 1: EXPLORE** (Plan Mode)
Claude reads files, trả lời câu hỏi, **không sửa gì**.

```
> read /src/auth and understand how we handle sessions and login.
> also look at how we manage environment variables for secrets.
```

### **Phase 2: PLAN** (Plan Mode)
Yêu cầu plan chi tiết.

```
> I want to add Google OAuth. What files need to change?
> What's the session flow? Create a plan.
```

💡 **Pro tip**: Nhấn `Ctrl+G` để mở plan trong text editor → chỉnh sửa trực tiếp trước khi proceed.

### **Phase 3: IMPLEMENT** (Normal Mode)
Switch back sang Normal Mode, Claude code theo plan.

```
> implement the OAuth flow from your plan. Write tests for the
> callback handler, run the test suite and fix any failures.
```

### **Phase 4: COMMIT**
```
> commit with a descriptive message and open a PR
```

---

## 🤔 Khi nào NÊN dùng Plan Mode?

### ✅ Plan Mode hữu ích khi:
- Task modify **nhiều files**
- Bạn **không quen** codebase đang sửa
- Approach **chưa chắc chắn** (multiple options)
- Feature **lớn**, có nhiều edge cases
- Cần **stakeholder review** trước khi code

### ❌ Plan Mode là overhead khi:
- Fix typo / rename variable
- Thêm 1 log line
- Change đơn giản, 1 file
- **"If you could describe the diff in one sentence, skip the plan"** — Anthropic

---

## 🌟 Advanced Patterns

### Pattern 1: "Interview Me" để tạo SPEC.md
**Dùng cho features LỚN** (theo Anthropic official):

```
> I want to build [brief description].
> Interview me in detail using the AskUserQuestion tool.
> Ask about technical implementation, UI/UX, edge cases, concerns, tradeoffs.
> Don't ask obvious questions, dig into hard parts I might not have considered.
> Keep interviewing until we've covered everything, then write a complete spec to SPEC.md.
```

**Workflow**:
1. Session 1: Interview → SPEC.md
2. `/clear` hoặc start new session
3. Session 2: Execute với fresh context, chỉ reference SPEC.md

→ Lợi ích: Clean context focused entirely on implementation.

### Pattern 2: Plan Mode như Handoff Tool
Khi context đầy, vào Plan Mode:

```
> I just enabled plan mode. Bring over all of the context that you need
> for the next agent. The next agent will not have any other context,
> so you'll need to be pretty comprehensive.
```

Claude sẽ explore codebase, gather context, write plan chi tiết. Khi xong, bạn có 4 options:
```
❯ 1. Yes, clear context and auto-accept edits (shift+tab)
  2. Yes, auto-accept edits
  3. Yes, manually approve edits
  4. Type here to tell Claude what to change
```

**Option 1** = clears old context, start fresh với plan. Tuyệt vời để "defrag" long sessions.

### Pattern 3: Multi-Agent Plan Review
1. Agent 1 (Plan Mode): Tạo plan
2. Agent 2 (fresh session): Review plan cho edge cases, security, patterns
3. Agent 3: Execute plan đã được review

---

## 🎨 Plan Mode + Thinking

Plan Mode hoạt động tốt với **extended thinking**:
- Ctrl+O để toggle verbose mode, xem reasoning
- Phrases không "thinking": "think", "think hard", "think more" chỉ là regular instructions (không allocate thinking tokens)
- Models với effort use **adaptive reasoning**: tự quyết định think bao nhiều

---

## 💡 Practical Tips

### 1. **Stay in Plan Mode để Chat an toàn**
- Có thể dùng Plan Mode để Claude trả lời câu hỏi về codebase mà không lo sửa nhầm
- Prompt: "Explain how X works" → Claude chỉ đọc, không edit

### 2. **Edit Plan trước khi Execute**
- Ctrl+G mở plan ở editor
- Remove items không cần, clarify ambiguous items
- Plan là **living document** - sửa được trước khi execute

### 3. **Kết hợp với Subagents trong Plan**
Plan có thể include việc dùng subagents:
```
Plan:
1. [Subagent] Investigate existing auth patterns
2. [Main] Implement OAuth based on findings
3. [Subagent] Review implementation for security
4. [Main] Fix issues, commit
```

### 4. **Plan cho Risky Refactoring**
- Plan Mode làm refactoring an toàn hơn nhiều
- Claude list toàn bộ files affected trước khi đụng gì
- Bạn có thể veto parts of the plan

### 5. **Auto Mode vs Plan Mode**
Khi config có auto mode:
- `useAutoModeDuringPlan: true` (default) → plan mode dùng auto mode semantics
- Config ở `/config` → "Use auto mode during plan"

---

## 🚫 Common Plan Mode Mistakes

### 1. **Quá Rely vào Plan**
Plan không thay thế verification. Plan xong vẫn phải:
- Write tests
- Run tests
- Verify output

### 2. **Không Iterate trên Plan**
Plan đầu thường không hoàn hảo. Ask: "What's missing? What edge cases did we skip?"

### 3. **Plan Mode cho Task Nhỏ**
Typo fix → Plan Mode = waste time. Direct prompt is fine.

### 4. **Không chuyển sang Normal Mode để Execute**
Forget to `Shift+Tab` back → Claude vẫn không edit được → frustration.

### 5. **Plan có sẵn nhưng không tạo SPEC.md**
Với task lớn, plan trong-conversation dễ mất. **Write to `SPEC.md`** để persist.

---

## 📐 Template Plan Mode Prompt cho Beginners

### Template 1: Codebase Onboarding
```
Enter plan mode.
1. Read the README and package.json
2. Explore the main directories in src/
3. Identify the key abstractions and patterns
4. Create a document called CODEBASE_NOTES.md with:
   - High-level architecture
   - Key files to know about
   - Main conventions
Don't modify any code files, only create the notes file.
```

### Template 2: Feature Planning
```
Plan mode. I want to add [FEATURE].
1. Research how similar features are implemented in this codebase
2. Identify all files that need changes
3. List the changes per file
4. Highlight risks and edge cases
5. Suggest tests to write
Create the plan as markdown, don't code yet.
```

### Template 3: Bug Investigation
```
Plan mode. Users report: [SYMPTOM].
1. Search code for relevant files
2. Trace the logic path
3. Hypothesize root cause with evidence
4. List files that would need to change
5. Suggest tests that would reproduce the bug
Don't fix yet, just investigate.
```

### Template 4: Refactor Planning
```
Plan mode. I want to refactor [X] to [Y].
1. Map out all usages of [X] in the codebase
2. Group them by difficulty/risk
3. Suggest a phased migration plan
4. Identify what tests we need BEFORE refactoring
5. Flag any breaking changes
```

---

## 📊 Quick Reference

| Action | In Plan Mode? | Notes |
|--------|---------------|-------|
| Read files | ✅ | Free to explore |
| Grep/search | ✅ | Read-only operations |
| Create plan docs | ✅* | Can write plan as file (vd SPEC.md) - needs verification |
| Edit existing files | ❌ | Must exit Plan Mode |
| Run tests | ⚠️ | Depends on permissions |
| `gh` CLI queries | ✅ | Read operations OK |

---

## 📚 Sources
- Anthropic Official Best Practices: https://code.claude.com/docs/en/best-practices (section "Explore first, then plan, then code")
- Common Workflows docs: https://code.claude.com/docs/en/common-workflows (section "Plan Mode")
- `ykdojo/claude-code-tips` Tip 8 (handoff), Tip 39 (plan + prototype)
- Reddit r/ClaudeAI discussions on plan mode workflows
