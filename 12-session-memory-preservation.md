# Topic 12: Giúp Claude Code KHÔNG QUÊN Những Điều Quan Trọng Khi Tạo Session Mới

> **Nguồn**: Anthropic Official blog, GitHub issues, `ykdojo/claude-code-tips` Tip 8, Reddit r/ClaudeAI

---

## Vấn Đề Cốt Lõi

Claude Code **không có memory tự động** giữa sessions (trừ CLAUDE.md files). Khi bạn:
- Close terminal
- Chạy `/clear`
- Bị auto-compact
- Start new conversation

→ Claude **mất hết context** về task hiện tại, decisions, failed approaches.

**Hậu quả**: Phải re-explain, Claude lặp lại mistakes cũ, waste tokens.

---

## 5 Cơ Chế Preservation (theo mức độ từ LIGHT → HEAVY)

### 1. **CLAUDE.md** (Always-loaded Memory)
**Dùng cho**: Context luôn đúng cho mọi session của project.

- Load **mọi session** tự động
- Quy tắc: Chỉ put vào đây những gì **không đổi** qua sessions
- Target: < 100 lines, ruthlessly pruned

**Best practices**:
```markdown
# Project Context
- Stack: Next.js 15, PostgreSQL, Prisma
- Auth: NextAuth + Google provider

# Non-obvious conventions
- All API responses MUST include `requestId`
- NEVER modify `schema.prisma` without migration plan

# Commands
- `npm test`: unit tests
- `npm run e2e`: integration tests
```

️ Anti-pattern: Put task-specific info vào CLAUDE.md → bloat → Claude ignore half.

---

### 2. **Custom Skills** (On-demand Memory)
**Dùng cho**: Workflows/knowledge cần **đôi khi**, không phải mỗi session.

- Đặt `.claude/skills/<name>/SKILL.md`
- Claude **tự invoke** khi relevant, hoặc user gọi `/<name>`
- **Không load** mỗi session → tiết kiệm tokens

**Example**: Skill cho fix GitHub issues
```markdown
---
name: fix-issue
description: Fix a GitHub issue
---
Analyze and fix the GitHub issue: $ARGUMENTS.
1. Use `gh issue view` to get details
2. Search codebase for relevant files
3. Implement changes
4. Write tests
5. Create descriptive commit, push, create PR
```

---

### 3. **HANDOFF.md** (Session-Specific Memory) 
**Dùng cho**: Context đang-làm-dở giữa sessions.

**Đây là technique quan trọng nhất cho continuity.**

#### Cách tạo (prompt template):
```
Put the rest of the plan in HANDOFF.md. Explain:
- What you tried
- What worked
- What didn't work
- Current state of the task
- Next concrete steps

So the next agent with fresh context is able to load THIS FILE ALONE
and nothing else to continue and finish the task.
```

#### Cách load session mới:
```
> HANDOFF.md
```
(Literally chỉ paste path - Claude đọc file, hiểu context, continue.)

#### Pattern nâng cao: Handoff Prompt Template
```markdown
We're continuing an ongoing task. Use this handoff as the current source of truth.

## Goal
[What I'm trying to achieve]

## Current Status
- Done: [completed items]
- In progress: [current item]
- To do: [remaining items]

## Important Context
- Audience/tone/constraints
- Key files: @src/auth/..., @docs/spec.md
- Deadline/success criteria

## Decisions Made
- [Decision + short reason]

## What to Avoid
- [Failed attempts, wrong directions, out-of-scope items]

## Open Questions
- [Unresolved items]

## Next Best Step
- [Concrete next action]

## How to Respond
- Start with short summary of understanding
- Continue from here, don't restart
- Ask only minimum needed questions
```

---

### 4. **SPEC.md** (Feature-level Memory)
**Dùng cho**: Features lớn trải qua nhiều sessions.

**Workflow** (Anthropic official):
1. Session 1: Plan Mode, ask Claude interview you → write `SPEC.md`
  ```
  I want to build [X]. Interview me using AskUserQuestion tool.
  Cover: implementation, UI/UX, edge cases, tradeoffs.
  Write complete spec to SPEC.md.
  ```
2. `/clear` → fresh session
3. Session 2: `implement SPEC.md` - clean context, focused

Lợi ích: Plan stable, implementation session không bị distracted bởi planning discussion.

---

### 5. **Plan Mode Handoff** (Fast Context Transfer)
**Dùng cho**: Switch agent giữa conversation.

```
> [Enter Plan Mode với Shift+Tab]
> I just enabled plan mode. Bring over all context needed for next agent.
> The next agent will have NO other context, so be comprehensive.
```

Claude explore codebase, gather context, write plan. Xong có 4 options, chọn:
```
1. Yes, clear context and auto-accept edits ← Dùng cái này
```

→ Auto `/clear` với plan làm context mới. Cực fast.

---

## Decision Tree: Chọn Cơ Chế Nào?

```
Task cần preserve?
 Global cho MỌI project → ~/.claude/CLAUDE.md
 Global cho 1 project, luôn relevant → ./CLAUDE.md (commit vào git)
 Project-specific nhưng chỉ bạn dùng → ./CLAUDE.local.md (gitignore)
 Occasional workflow/knowledge → ./claude/skills/<name>/SKILL.md
 Current task in-progress → HANDOFF.md
 Feature đang implement → SPEC.md
 Mid-session agent switch → Plan Mode "bring over context"
```

---

## Advanced Techniques

### Technique A: **Auto-generate Handoff với Hook**
Tạo hook tự động suggest `/half-clone` khi context > 85%:

```json
{
 "hooks": {
  "Stop": [{
   "hooks": [{
    "type": "command",
    "command": "~/.claude/scripts/check-context.sh"
   }]
  }]
 }
}
```

(Script từ `ykdojo/claude-code-tips`)

### Technique B: **Resume Conversations**
```bash
claude --continue   # Resume most recent
claude --resume    # Pick from recent list
claude --fork-session # Resume + branch off
```

Rename sessions với `/rename "payment-integration"` để find lại dễ dàng.

### Technique C: **Periodic Checkpoints trong Long Tasks**
Mỗi major milestone, ask Claude update HANDOFF.md:
```
> Update HANDOFF.md to reflect progress. Add what we completed in this session.
```

Benefit: Nếu crash, resume từ latest checkpoint, không từ start.

### Technique D: **Session-specific Notes File**
Create `session-notes-YYYY-MM-DD.md` for each major session:
- Decisions made
- Learnings
- Anti-patterns discovered
→ Reference trong future CLAUDE.md updates.

### Technique E: **Git Commits as Memory**
- Commit frequently với descriptive messages
- Claude có thể `git log --oneline -20` để hiểu recent work
- Commit messages = auto-generated history

### Technique F: **Project Journals**
Tạo `docs/decisions.md` (ADR - Architecture Decision Records):
- Why chose X over Y
- Context at time of decision
- Trade-offs accepted
→ Claude read khi cần context history.

---

## Pro Workflow: Multi-Session Feature Implementation

### Example: Build OAuth flow trong 3 sessions

**Session 1 (Research + Plan)**:
```
> [Plan Mode]
> Interview me about adding Google OAuth.
> After full interview, write SPEC.md with:
> - Architecture decisions
> - File changes needed
> - Edge cases
> - Testing strategy
```
End session, SPEC.md saved.

**Session 2 (Implementation)**:
```
> Read SPEC.md. Implement Phase 1 (auth middleware).
> Write tests. Run them. Fix failures.
> When Phase 1 complete, update SPEC.md status.
```
At end:
```
> Update SPEC.md + write HANDOFF.md for Phase 2.
```

**Session 3 (Continue)**:
```
> HANDOFF.md
```
Claude reads → continues Phase 2.

---

## Common Mistakes

### 1. **Đổ Task-specific Info vào CLAUDE.md**
```
 # Currently working on payment integration with Stripe
 # TODO: Fix the token refresh bug
```
→ Làm CLAUDE.md bloat, Claude ignore actual rules.
→ Put vào HANDOFF.md instead.

### 2. **Không Write HANDOFF.md Trước `/clear`**
`/clear` → mất hết. Luôn HANDOFF trước khi clear task đang dở.

### 3. **HANDOFF.md Quá Sơ Sài**
```
 "Working on auth. Need to add OAuth."
```
→ Next session không biết: Google/GitHub? Files đã touch? Approaches thất bại?

Good HANDOFF:
```markdown
## Status
- Added passport-google-oauth20 dependency
- Created `/auth/google/callback` route (file: src/routes/auth.ts)
- Tried to use passport.session() - doesn't work with our JWT setup
- Currently: implementing JWT-based session from OAuth response

## Files Changed
- src/routes/auth.ts (callback handler, line 45-80)
- src/middleware/auth.ts (added JWT generation)

## Next
- Add rate limiting to /auth/google endpoint
- Write integration test for full OAuth flow
```

### 4. **Không Review Handoff Trước Khi Tin**
Claude có thể miss details. Quick review + add bổ sung:
```
> Did you include the note about the JWT secret rotation?
```

### 5. **Commit HANDOFF.md vào Git**
HANDOFF.md là **session-specific**, personal. Put vào `.gitignore`:
```
.claude/handoff*.md
HANDOFF.md
```

---

## Templates Ready-to-Use

### Template 1: Simple Handoff (end of session)
```
> Before we stop, write HANDOFF.md summarizing:
> 1. What we accomplished
> 2. What's still in progress
> 3. Known issues/blockers
> 4. The exact next step
> Be specific with file paths and line numbers where relevant.
```

### Template 2: Handoff with Verification
```
> Write HANDOFF.md. After writing, READ it back and check:
> - Does it include all files we modified?
> - Does it mention the 3 approaches we ruled out?
> - Is the next step concrete (not vague)?
> Fix any gaps before finishing.
```

### Template 3: Cross-session CLAUDE.md Update
```
> Looking at this session's learnings, what belongs in CLAUDE.md
> (project-wide, always-relevant rules)?
> Distinguish from session-specific items (those go in HANDOFF.md).
> Suggest specific CLAUDE.md edits.
```

---

## Sources
- Anthropic blog "Using Claude Code: session management": https://claude.com/blog/using-claude-code-session-management-and-1m-context
- GitHub Issue #11455 (session handoff feature request): https://github.com/anthropics/claude-code/issues/11455
- Handoff Prompt Guide: https://www.jdhodges.com/blog/ai-session-handoffs-keep-context-across-conversations/
- `ykdojo/claude-code-tips` Tip 8 (handoff) + dx plugin
- Anthropic Best Practices (SPEC.md workflow): https://code.claude.com/docs/en/best-practices
