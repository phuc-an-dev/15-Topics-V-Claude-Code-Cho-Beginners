# Topic 10: Best Practices Using Agents (Subagents)

> **Nguồn**: Anthropic official docs + blog, Medium (Rick Hightower, Sathish Raju), MindStudio workflow patterns, claudefa.st sub-agent guide

---

## Subagent Là Gì?

**Subagent** = Specialized AI assistant trong Claude Code, có:
- **Context window riêng** (isolated)
- **System prompt custom** (chuyên về 1 domain)
- **Tool access limited** (chỉ tools cần thiết)
- **Permission riêng**

### Tại sao dùng subagent?
> "Subagents are one of the **most powerful tools available**... Since context is your fundamental constraint." — Anthropic

**Problem without subagents**:
- Main conversation đầy files đã đọc, logs, search results
- Không cần reference lại → waste tokens
- Performance degrades

**Solution with subagents**:
- Side task trong context riêng
- **Chỉ return summary** vào main conversation
- Main conversation stay clean

---

## Khi NÀO Nên Dùng Subagent?

### Dùng subagent khi:

1. **Investigation/Research**
  - Đọc hàng chục files
  - Understand pattern trước khi code
  - Security review, performance audit

2. **Parallel Independent Tasks**
  - Update cùng pattern trong nhiều files
  - Fix errors across several files
  - Chạy multiple audits cùng lúc

3. **Fresh Perspective cho Review**
  - Code review (không bias bởi implementation đã viết)
  - Test review
  - Security audit

4. **Repeated Workflows**
  - Cùng loại task lặp đi lặp lại → define custom subagent

5. **Cost Optimization**
  - Route simple tasks → cheaper models (Haiku/Sonnet)
  - Main session on Opus, subagents on Sonnet

### KHÔNG dùng subagent khi:

- Task đơn giản, 1-2 files
- Sequential task phụ thuộc nhiều vào context main conversation
- Task < 5 phút làm trực tiếp nhanh hơn

---

## ️ Built-in Subagents

Claude Code có sẵn:
- **Explore**: Fast, read-only code search
- **Plan**: Planning mode (tương đương Shift+Tab Plan Mode)
- **general-purpose**: Generic delegation

---

## ️ Cách Tạo Custom Subagent

### Method 1: Interactive (dễ nhất)
```bash
/agents
```
Guided setup, generate first draft từ description.

### Method 2: Manual Markdown file

**Location**:
- Project: `.claude/agents/<name>.md` (shared via git)
- User: `~/.claude/agents/<name>.md` (global)

**Template**:
```markdown
---
name: security-reviewer
description: Reviews code changes for security vulnerabilities, injection risks, auth issues, and sensitive data exposure. Use PROACTIVELY before commits touching auth, payments, or user data.
tools: Read, Grep, Glob
model: sonnet
---

You are a security-focused code reviewer. Analyze changes for:
- SQL injection, XSS, command injection risks
- Authentication and authorization gaps
- Sensitive data in logs or responses
- Insecure defaults

Provide specific line references and suggested fixes.
```

### Key Fields

| Field | Purpose |
|-------|---------|
| `name` | Unique identifier (kebab-case) |
| `description` | Claude dùng này để decide khi nào delegate |
| `tools` | Subset of tools (Read, Grep, Glob, Edit, Bash, WebFetch, ...) |
| `model` | `haiku`, `sonnet`, or `opus` (default: sonnet) |

### Pro tip for auto-delegation
Thêm **"PROACTIVELY"** hoặc **"MUST BE USED"** trong description:
> "Use PROACTIVELY for security review before any commit."

→ Claude aggressive delegate to this agent.

---

## Common Subagent Patterns

### Pattern 1: Explore-Plan-Execute Pipeline 

**3-phase pattern** (Sathish Raju) cho complex engineering tasks:

```
1. Explore Agent (read-only)
  → Scan codebase, understand architecture
  → Output: explored-findings.md

2. Plan Agent (analytical)
  → Design approach based on findings
  → Output: implementation-plan.md
  ← HUMAN REVIEW GATE ←

3. Execute Agent (write access)
  → Implement theo plan
  → Output: code + tests
```

**Key insight**: Human review gate giữa Plan và Execute, KHÔNG giữa Explore và Plan.
- Exploration = cheap, let agent read freely
- Planning = analytical, let agent design
- Before file modifications → approve plan

### Pattern 2: Hub-and-Spoke (Orchestrator-Workers)

```
     Main Agent (Orchestrator)
     /  |  |  \
   Agent Agent Agent Agent
   (A)  (B)  (C)  (D)
```

Main agent delegates tasks to specialized workers, synthesizes results.

**From Anthropic's research**: Claude Code primarily uses **orchestrator-workers pattern**.

### Pattern 3: Split-and-Merge (Parallel Independent)

```
Task → Split → Agent1, Agent2, Agent3 (parallel) → Merge
```

**Example**: Document 50 functions
- Split: 10 batches of 5 functions
- Run 10 subagents simultaneously
- Merge: Combine all docstrings

### Pattern 4: Writer/Reviewer Pattern

```
Agent A (Writer): Implement feature
   ↓
Agent B (Reviewer): Fresh session, review code critically
   ↓
Agent A: Address review feedback
```

**Why fresh session helps**: No bias toward code just written.

### Pattern 5: Test-First Pipeline (Multi-agent TDD)

```
 tdd-test-writer → tdd-implementer → tdd-refactorer
```

Mỗi agent có context riêng, không bleed implementation vào tests.

---

## Example Custom Subagents

### Subagent 1: `code-reviewer`
```markdown
---
name: code-reviewer
description: Reviews code for bugs, logic errors, and pattern consistency with CLAUDE.md rules.
tools: Read, Grep, Glob, Bash
model: opus
---

You are a senior engineer reviewing PRs. Look for:
- Logic errors, edge cases
- CLAUDE.md rule violations
- Performance issues
- Security risks

For each issue: provide file:line reference, explanation, suggested fix.
Score confidence 0-100. Only report issues ≥80 confidence.
```

### Subagent 2: `test-writer`
```markdown
---
name: test-writer
description: Writes comprehensive tests covering happy path + edge cases. Use PROACTIVELY after implementing any feature.
tools: Read, Write, Edit, Bash
---

You are a test specialist. For given feature:
- Identify happy path, edge cases, error conditions
- Write tests BEFORE implementation (TDD)
- Ensure tests fail initially
- After implementation, verify all pass
- Aim for meaningful coverage, not just high %
```

### Subagent 3: `debugger`
```markdown
---
name: debugger
description: Systematic debugging. Use when encountering bug, test failure, or unexpected behavior BEFORE proposing fixes.
tools: Read, Grep, Glob, Bash
---

You are a debugging specialist. Process:
1. Reproduce the issue
2. Form hypothesis with evidence
3. Isolate the cause (don't band-aid)
4. Explain ROOT cause, not symptom
5. Propose minimal fix addressing root
6. Suggest tests to prevent regression
```

### Subagent 4: `docs-writer`
```markdown
---
name: docs-writer
description: Generates and maintains documentation.
tools: Read, Write, Edit
model: haiku
---

Write clear, concise documentation:
- API endpoints: request/response format, errors, examples
- Functions: purpose, params, returns, gotchas
- Keep sync with code changes
- Include working examples
```

---

## Agent Usage Rules trong CLAUDE.md

Từ GitHub gist (tomas-rampas):

```markdown
## Agent Usage Rules
- Use `plan-agent` for high-level planning, architecture decisions
- Use `reader-agent` for file reading and analysis
- Use `maker-agent` for code writing, editing, refactoring
- Use `security-agent` for security analysis
- Use `test-agent` for testing, validation, QA
- Use `docs-agent` for documentation
- Use `debug-agent` for debugging, error investigation

## Domain Parallel Patterns
When implementing cross-domain features, spawn PARALLEL agents:
- **Frontend agent**: React components, UI state, forms
- **Backend agent**: API routes, business logic
- **Database agent**: Schema, migrations, queries
Each agent owns their domain.

## Explore-Plan-Execute Pipeline
For ANY task touching >3 files OR involving refactoring:
1. Explore agent: Scan codebase
2. Plan agent: Design approach → HUMAN REVIEW
3. Execute agent: Implement
```

---

## Cost Optimization

### Use `CLAUDE_CODE_SUBAGENT_MODEL` env var
```bash
# Main on Opus, subagents on Sonnet
export CLAUDE_CODE_SUBAGENT_MODEL="claude-sonnet-4-5-20250929"
```

### Model selection theo task
| Task | Model | Reason |
|------|-------|--------|
| High-level planning | Opus | Complex reasoning |
| Code implementation | Sonnet | Balance speed/quality |
| Simple analysis | Haiku | Fast, cheap |
| Documentation | Haiku | Verbose text generation |
| Security review | Opus | Depth matters |

---

## Subagents vs Agent Teams

### Subagents (single session)
- Hierarchical: Orchestrator → Workers
- All communication through orchestrator
- Good cho sequential tasks

### Agent Teams (across sessions)
- Peer-to-peer: Shared task list/state
- Agents read each other's outputs directly
- Good cho longer-running parallel work
- **~7x more tokens** than standard sessions!

**Rule**: Default to subagents. Agent teams cho specific use cases justifying the token cost.

---

## Safety with Subagents

### Anthropic's 3 safeguards
1. **Minimum necessary permissions** - tools field limits access
2. **Prefer reversible actions** - read-only subagents safer
3. **Human approval checkpoints** - for high-stakes decisions

### `--allowedTools` flag
```bash
claude -p "prompt" --allowedTools "Read,Grep,Bash(npm test:*)"
```
Restrict tools for automation workflows.

### Permission Modes
- **plan mode** (`--permission-mode plan`): Read-only, safe exploration
- **default mode**: Ask before destructive
- **auto mode**: Classifier-based approval
- **bypassPermissions**: ️ Only trong containers

---

## Common Anti-Patterns

### 1. **Over-parallelizing**
 Launch 10 parallel agents cho simple feature
 Group related micro-tasks. Parallel chỉ khi domains independent.

### 2. **Under-parallelizing**
 Run 4 independent analyses sequentially
 Spot domain independence, parallelize.

### 3. **Vague Invocations**
 "Use a subagent to implement the feature"
 Specific scope, file references, success criteria

### 4. **Orchestrator Context Full**
Khi orchestrator collect too many subagent results → context đầy → degrades.
 Subagents phải return **concise summaries**, not raw output.

### 5. **No Tool Restrictions**
Default: subagent có access mọi tools parent có.
 Explicitly limit với `tools:` field trong frontmatter.

### 6. **Subagent for Simple Tasks**
 Fix typo → spawn subagent
 Do inline. Subagents add overhead.

### 7. **Forget Structured Output**
Parallel subagents cần output schema consistent để merge.
 Define output format in system prompt.

---

## Invocation Checklist

Mỗi lần dispatch subagent, ensure có đủ:

- [ ] **Scope clear**: What exactly should agent do?
- [ ] **File references**: Which files to read/modify?
- [ ] **Success criteria**: How know when done?
- [ ] **Output format**: What to return?
- [ ] **Constraints**: What NOT to do?

### Template
```
Use the <agent-name> subagent.

Task: [Specific scope]
Files to reference: [list]
Success criteria: [verifiable]
Output format: [structured]
Constraints: [what not to do]
```

---

## Sources
- Anthropic Official: https://code.claude.com/docs/en/sub-agents
- Anthropic Blog: https://claude.com/blog/subagents-in-claude-code
- Anthropic Course: https://anthropic.skilljar.com/introduction-to-subagents
- Sathish Raju guide: https://medium.com/@sathishkraju/claude-code-subagents-the-complete-guide-to-ai-agent-delegation-d0a9aba419d0
- Rick Hightower: https://medium.com/@richardhightower/claude-code-subagents-and-main-agent-coordination-...
- MindStudio patterns: https://www.mindstudio.ai/blog/claude-code-agentic-workflow-patterns
- claudefa.st: https://claudefa.st/blog/guide/agents/sub-agent-best-practices
- GitHub gist (delegation rules): https://gist.github.com/tomas-rampas/a79213bb4cf59722e45eab7aa45f155c
