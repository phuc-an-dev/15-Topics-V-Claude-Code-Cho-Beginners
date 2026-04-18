# Topic 7: Best Concepts Khi Work Với Claude (Test First, ...)

> **Nguồn**: Anthropic official docs, Steve Kinney course, FlorianBruniaux ultimate guide, alexop.dev, Reddit r/ClaudeAI

---

## 🎯 Các Concepts Quan Trọng Nhất

Đây không phải tools, mà là **mindset & methodology** khi work với Claude.

---

## 1. **Test-First / TDD (Red-Green-Refactor)** ⭐ Quan trọng nhất

### Vấn đề
Claude **defaults to implementation-first**:
> "When I ask Claude to 'implement feature X', it writes the implementation first. Every time." — alexop.dev

→ Hậu quả: Claude viết "Happy Path", ignore edge cases, tests become afterthought.

### Giải pháp: Explicit TDD prompting

**3 phases Red-Green-Refactor**:

#### 🔴 Phase 1: RED - Write Failing Test
```
Write a FAILING test for [feature].
Do NOT write implementation yet.
Specify input/output pairs.
```

**Example**:
```
Let's use TDD. First, write failing tests for:
1. Shortening a URL returns a short code
2. Retrieving a short code returns original URL
3. Invalid URLs are rejected
4. Expired links return error

Do NOT implement anything yet.
```

#### 🟢 Phase 2: GREEN - Make Test Pass
```
Tests are written and failing.
Now implement the MINIMUM code to make them pass.
```

#### 🔵 Phase 3: REFACTOR - Improve
```
Tests pass. Now refactor the implementation for clarity/performance
WITHOUT changing test behavior. Tests must still pass.
```

### Advanced: Multi-Agent TDD (alexop.dev pattern)
Dùng skills + subagents để enforce strict TDD:
- `tdd-test-writer` subagent: chỉ viết failing tests
- `tdd-implementer` subagent: chỉ implement to pass tests
- `tdd-refactorer` subagent: chỉ refactor

→ Mỗi subagent có context riêng → **không bleed implementation logic vào tests**.

### Automate với Hooks
```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit|Write",
      "command": "npm test --watchAll=false 2>&1 | head -20"
    }]
  }
}
```
→ Auto-run tests sau mỗi edit.

---

## 2. **Explore → Plan → Code → Commit** ⭐

**4 phases workflow** (Anthropic official):

1. **Explore** (Plan Mode): Read files, understand codebase
2. **Plan** (Plan Mode): Create detailed implementation plan
3. **Code** (Normal Mode): Implement theo plan + write tests
4. **Commit**: Descriptive message, open PR

**Key insight**: "Letting Claude jump straight to coding can produce code that solves **the wrong problem**."

---

## 3. **Give Claude a Way to Verify** ⭐

> "This is the single **highest-leverage thing** you can do." — Anthropic

Without verification, Claude produces code that **looks right but doesn't work**.

### Verification strategies

| Type | Example |
|------|---------|
| Tests | "Run pytest, fix failures" |
| Screenshots | "[paste target] match this design" |
| Expected output | "Should return `[1, 2, 3]` for input `[3, 1, 2]`" |
| Linter | "Run eslint, fix warnings" |
| Type checker | "Run `tsc --noEmit`, no errors allowed" |
| Build | "Build must pass" |

### Prompt patterns
```
Implement [X]. After implementing:
1. Run tests
2. Fix any failures
3. Run linter
4. Show me the output of all verifications
```

---

## 4. **Address Root Causes, Not Symptoms**

### ❌ Bad prompt
> "The build is failing"

### ✅ Good prompt
> "The build fails with this error: [paste]. Fix it and verify build succeeds. **Address the root cause**, don't suppress the error."

### Why this matters
Claude có bias "quick fix":
- Catch exception và suppress → problem returns later
- Add `// @ts-ignore` → type error hidden
- Comment out failing test → false confidence

**Prompt technique**:
```
Before fixing, explain:
1. What's the ACTUAL root cause?
2. What's the simplest fix at root level?
3. What tests would prevent regression?
```

---

## 5. **Provide Specific Context** ⭐

### Rule: Claude can infer intent, but can't read your mind.

| ❌ Vague | ✅ Specific |
|---------|------------|
| "fix the login bug" | "users report login fails after session timeout. Check `src/auth/`, especially token refresh. Write failing test first." |
| "add tests for foo.py" | "test foo.py covering logged-out user edge case. Avoid mocks." |
| "make dashboard better" | "[paste screenshot] implement this design. Take screenshot of result, compare, fix differences." |

### Technique: Reference Existing Patterns
```
Look at how existing widgets are implemented on home page.
HotDogWidget.php is a good example.
Follow the pattern to implement a new calendar widget.
Build from scratch without libraries other than those already used.
```

---

## 6. **Course-Correct Early and Often**

### Principle: Tight feedback loops > long sessions with corrections.

### Tools
- **`Esc`**: Stop Claude mid-action, context preserved
- **`Esc + Esc` / `/rewind`**: Open rewind menu, restore previous state
- **`"Undo that"`**: Ask Claude revert
- **`/clear`**: Reset context between unrelated tasks

### Rule of Thumb
> "If you've corrected Claude **more than twice** on the same issue, context is cluttered. Run `/clear` and start with a more specific prompt." — Anthropic

---

## 7. **Manage Context Aggressively**

### Concept: Context is your fundamental constraint.

### Strategies
- **`/clear` frequently** between tasks
- **`/compact <instructions>`** để guide compaction
- **Subagents for investigation** - separate context
- **`/btw`** cho side questions (không vào history)
- **Customize compaction** trong CLAUDE.md

### When to Clear vs Compact

| Situation | Action |
|-----------|--------|
| New task | `/clear` |
| Mid-task, context full | `/compact <focus>` |
| Correcting 2+ times same issue | `/clear` + better prompt |
| Switching topic briefly | `/btw "quick question..."` |

---

## 8. **Use Subagents for Investigation**

### Concept: Subagents = separate context windows.

### Why it matters
Khi Claude investigate codebase, nó đọc nhiều files → main context đầy.
Subagents explore trong context riêng, **report back summary**.

### Example prompts
```
Use subagents to investigate how our authentication handles token refresh.
```
```
Use a subagent to review this code for edge cases.
```

### Use cases
- Codebase onboarding
- Security review
- Edge case audit
- Performance investigation
- Legacy code analysis

---

## 9. **"Interview Me First" Pattern**

### For larger features:
```
I want to build [brief description].
Interview me in detail using the AskUserQuestion tool.
Ask about technical implementation, UI/UX, edge cases, concerns, tradeoffs.
Don't ask obvious questions, dig into hard parts I might not have considered.
Keep interviewing until we've covered everything.
Then write a complete spec to SPEC.md.
```

**Why this works**:
- Surfaces assumptions
- Catches edge cases early
- Forces thinking before coding
- Creates persistent artifact (SPEC.md)

---

## 10. **Let Claude Ask Questions Like a Senior Engineer**

### Concept: Claude Code as onboarding tool.

Ask questions như với senior engineer:
- How does logging work?
- How do I make a new API endpoint?
- What does `async move { ... }` do on line 134 of `foo.rs`?
- What edge cases does `CustomerOnboardingFlowImpl` handle?
- Why does this code call `foo()` instead of `bar()` on line 333?

→ Effective onboarding workflow, improves ramp-up time.

---

## 11. **The 2-Strike Rule**

> "Only add a note the **second time** you have to correct Claude on the same thing (first-time issues are usually one-offs)." — token guide

### Application
- First mistake: Just correct it
- Second same mistake: Add to CLAUDE.md
- Patterns bạn lặp 3+ lần → convert sang Skill or Hook

---

## 12. **Developer Skills Matter**

### Concept: AI doesn't replace problem-solving.

> "Your problem-solving skills and software engineering skills are still **highly relevant** in the world of agentic coding." — ykdojo

### What still matters:
- Breaking down problems
- Knowing `git bisect` exists
- Understanding when to use tmux, Playwright, Docker
- Architecture patterns
- Debugging intuition

**Pattern**: Software engineering knowledge + Claude Code = exponential productivity.

---

## 13. **Choose the Right Level of Abstraction**

### Spectrum từ Vibe Coding → Deep Dive

| Level | When to use |
|-------|-------------|
| **Vibe coding** (fly over iceberg) | One-time projects, non-critical code, rapid prototyping |
| **Structural review** (swim around) | Feature development, normal refactoring |
| **Line-by-line** (dive deep) | Critical paths, security-sensitive, production |

### Rule: Not binary. You control the zoom level.

**Prompts to zoom in**:
- "Explain this line"
- "Why did you choose X over Y?"
- "What edge cases does this handle?"

**Prompts to zoom out**:
- "Just make it work, I'll review later"
- "Quick prototype, don't worry about optimization"

---

## 14. **Be Braver in the Unknown**

> "Since I started using Claude Code more intensely, I became more **brave in the unknown**." — ykdojo

### Pattern
- Not expert in React? → Dig into React codebase với Claude
- Never used Rust? → Start Rust project với Claude
- Don't know Kubernetes? → Deploy với Claude's help

**Process**:
1. Ask questions iteratively
2. Experiment
3. Some dead ends → try different direction
4. Control pace (fast để explore, slow để understand)

---

## 15. **Automation of Automation** (Meta-principle)

### Evolution path
1. **Manual** copy-paste và run commands
2. **CLAUDE.md** - stop repeating instructions
3. **Skills** - package workflows
4. **Hooks** - deterministic automation
5. **Custom scripts** - automate entire pipelines

### Question to ask yourself
> "Am I doing this **3+ times**? If yes, automate."

---

## 🎯 Quick Reference - Core Concepts Checklist

### Before Coding
- [ ] Test-first (if applicable)
- [ ] Clear verification criteria defined
- [ ] Plan mode for complex features
- [ ] Specific context in prompts

### During Coding
- [ ] Tight feedback loops (correct early)
- [ ] Root cause fixes, not symptoms
- [ ] Subagents for investigation
- [ ] Reference existing patterns

### After Coding
- [ ] Tests pass (auto-verify)
- [ ] Linter clean
- [ ] Type check clean
- [ ] Ask: "Any edge cases missed?"
- [ ] Commit with descriptive message

### Session Hygiene
- [ ] `/clear` between unrelated tasks
- [ ] `/compact` when mid-task gần đầy
- [ ] Handoff doc khi session dài
- [ ] 2-strike rule cho CLAUDE.md updates

---

## 📚 Resources
- Anthropic Best Practices: https://code.claude.com/docs/en/best-practices
- Steve Kinney TDD course: https://stevekinney.com/courses/ai-development/test-driven-development-with-claude
- alexop.dev Custom TDD Workflow: https://alexop.dev/posts/custom-tdd-workflow-claude-code-vue/
- FlorianBruniaux/claude-code-ultimate-guide (GitHub)
- ykdojo/claude-code-tips (Tips 3, 9, 22, 34, 35)
