# Topic 13: Best Hooks Cho Developer

> **Nguồn**: Anthropic official docs, `disler/claude-code-hooks-mastery`, ClaudeLog, Pixelmojo, Pasquale Pillitteri, techsy.io, datacamp

---

## Hooks Là Gì?

**Hooks = Deterministic automation scripts** chạy ở specific lifecycle events trong Claude Code.

### Tại sao quan trọng?
> "Claude is a language model - it operates on **probabilities**. Your context engineering can nudge behavior, but it **can't guarantee** it. **Hooks bypass the LLM entirely**." — techsy.io

**Hooks VS CLAUDE.md**:
- CLAUDE.md = **advisory** (Claude có thể ignore)
- Hooks = **deterministic** (luôn chạy, không miss)

---

## 21 Lifecycle Events (focus 10 quan trọng nhất)

### Tool Events 
1. **`PreToolUse`** - Trước khi tool run (có thể block/modify)
2. **`PostToolUse`** - Sau khi tool xong successful
3. **`PostToolUseFailure`** - Khi tool fail (context cho Claude)
4. **`PermissionRequest`** - Trước khi show permission dialog
5. **`PermissionDenied`** - Khi auto mode deny

### Session Events
6. **`SessionStart`** - Session start/resume/clear
7. **`SessionEnd`** - Session terminate
8. **`UserPromptSubmit`** - User submit prompt
9. **`Stop`** - Claude finish responding
10. **`SubagentStop`** - Subagent finish

### Rule of thumb
> "You'll use **PreToolUse và PostToolUse** for 80% of your hooks. **SessionStart** is next most useful." — techsy.io

---

## ️ 4 Hook Handler Types

| Type | Purpose | % Usage |
|------|---------|---------|
| **command** (shell) | 90% cases: format, lint, validate | Most common |
| **http** (POST URL) | External integrations (Slack, Webhooks) | Integration |
| **prompt** (LLM yes/no) | Context-dependent decisions | Nuanced control |
| **agent** (spawn subagent) | Deep verification với tool access | Complex logic |

---

## 10 Best Hooks Cho Developer (Ready-to-Use)

### 1. **Auto-Format After Edits** (must-have)
Prettier sau mỗi file write:

```json
{
 "hooks": {
  "PostToolUse": [{
   "matcher": "Write|Edit|MultiEdit",
   "hooks": [{
    "type": "command",
    "command": "npx prettier --write \"$CLAUDE_TOOL_INPUT_FILE_PATH\""
   }]
  }]
 }
}
```

**Variations**:
- Python: `black` hoặc `ruff format`
- Go: `gofmt`
- Rust: `cargo fmt`
- Java: `mvn spotless:apply`

### 2. **Block Dangerous Commands** ️ (security)
```json
{
 "hooks": {
  "PreToolUse": [{
   "matcher": "Bash",
   "hooks": [{
    "type": "command",
    "command": "echo \"$CLAUDE_TOOL_INPUT\" | grep -qE 'rm -rf /|DROP TABLE|:(){ :|:' && exit 2 || exit 0"
   }]
  }]
 }
}
```

Chặn: `rm -rf /`, `DROP TABLE`, fork bombs, etc.

️ **Critical**: `exit 2` = block (không phải `exit 1`). Đây là mistake phổ biến nhất.

### 3. **Auto-Run Tests After Code Changes**
```json
{
 "hooks": {
  "PostToolUse": [{
   "matcher": "Edit|Write",
   "hooks": [{
    "type": "command",
    "command": "npm test 2>&1 | tail -20",
    "timeout": 60000
   }]
  }]
 }
}
```

### 4. **Type Check After TSX Edits** (React/TS devs)
```json
{
 "hooks": {
  "PostToolUse": [{
   "matcher": "Edit|Write",
   "if": "Edit(**/*.tsx)|Write(**/*.tsx)",
   "hooks": [{
    "type": "command",
    "command": "npx tsc --noEmit 2>&1 | head -30"
   }]
  }]
 }
}
```

### 5. **Inject Git Context on Session Start** 
```json
{
 "hooks": {
  "SessionStart": [{
   "hooks": [{
    "type": "command",
    "command": "echo '{\"additionalContext\": \"Branch: '$(git branch --show-current)'\\nRecent commits:\\n'$(git log --oneline -5)'\"}'"
   }]
  }]
 }
}
```

Claude bắt đầu session biết ngay branch hiện tại + recent commits.

### 6. **Desktop Notification Khi Claude Xong** 
```json
{
 "hooks": {
  "Stop": [{
   "hooks": [{
    "type": "command",
    "command": "terminal-notifier -message 'Claude finished' -title 'Claude Code' -sound default"
   }]
  }]
 }
}
```

**Game-changer**: Kick off task → làm việc khác → notification khi xong.

Linux alternative: `notify-send "Claude finished"`

### 7. **Block Writes vào Protected Paths**
```json
{
 "hooks": {
  "PreToolUse": [{
   "matcher": "Edit|Write",
   "hooks": [{
    "type": "command",
    "command": "PATH=$(echo \"$CLAUDE_TOOL_INPUT\" | jq -r '.file_path'); if echo \"$PATH\" | grep -qE '(migrations/|\\.env|secrets/)'; then echo 'Blocked: protected path' >&2; exit 2; fi; exit 0"
   }]
  }]
 }
}
```

### 8. **Auto-Suggest Half-Clone Khi Context > 85%**
Từ `ykdojo/claude-code-tips`:
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

Script check context %, suggest `/half-clone` nếu > 85%.

️ Requires `auto-compact` disabled (otherwise conflicts).

### 9. **Pre-Commit Backup**
```json
{
 "hooks": {
  "PreToolUse": [{
   "matcher": "Bash",
   "if": "Bash(git commit:*)",
   "hooks": [{
    "type": "command",
    "command": "git stash push -m \"pre-claude-commit-$(date +%s)\" --keep-index || true"
   }]
  }]
 }
}
```

### 10. **Log Every Session to File** 
```json
{
 "hooks": {
  "Stop": [{
   "hooks": [{
    "type": "command",
    "command": "echo \"$(date): session completed\" >> ~/claude-work.log",
    "async": true
   }]
  }]
 }
}
```

Async = không block Claude từ stopping.

---

## Starter Pack Config (All-in-One)

Copy vào `~/.claude/settings.json`:

```json
{
 "hooks": {
  "PreToolUse": [
   {
    "matcher": "Bash",
    "hooks": [{
     "type": "command",
     "command": "echo \"$CLAUDE_TOOL_INPUT\" | grep -qE 'rm -rf /|DROP TABLE' && { echo 'Dangerous command blocked' >&2; exit 2; } || exit 0"
    }]
   },
   {
    "matcher": "Edit|Write",
    "hooks": [{
     "type": "command",
     "command": "FILE=$(echo \"$CLAUDE_TOOL_INPUT\" | jq -r '.file_path'); echo \"$FILE\" | grep -qE '\\.env|secrets/' && { echo 'Protected path blocked' >&2; exit 2; } || exit 0"
    }]
   }
  ],
  "PostToolUse": [
   {
    "matcher": "Write|Edit|MultiEdit",
    "hooks": [{
     "type": "command",
     "command": "FILE=$(echo \"$CLAUDE_TOOL_INPUT\" | jq -r '.file_path'); case \"$FILE\" in *.ts|*.tsx|*.js|*.jsx) npx prettier --write \"$FILE\" 2>&1 ;; *.py) ruff format \"$FILE\" 2>&1 ;; esac"
    }]
   }
  ],
  "SessionStart": [
   {
    "hooks": [{
     "type": "command",
     "command": "echo '{\"additionalContext\": \"Branch: '$(git branch --show-current 2>/dev/null)'\"}'"
    }]
   }
  ],
  "Stop": [
   {
    "hooks": [{
     "type": "command",
     "command": "command -v terminal-notifier >/dev/null && terminal-notifier -message 'Claude finished' -title 'Claude Code' 2>/dev/null || true",
     "async": true
    }]
   }
  ]
 }
}
```

---

## Advanced Patterns

### Pattern 1: **PreToolUse Input Modification** (v2.0.10+)
Hooks có thể **rewrite tool inputs** thay vì block:

```bash
#!/bin/bash
INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command')

# Convert relative paths to absolute
if echo "$COMMAND" | grep -q "^cd "; then
 # Modify and output new JSON
 MODIFIED=$(echo "$INPUT" | jq '.tool_input.command |= sub("\\./"; "'"$PWD"'/")')
 echo "$MODIFIED"
fi
exit 0
```

Use cases: Path normalization, auto-inject env vars, redact secrets, add `--dry-run` to risky commands.

### Pattern 2: **Skill-Embedded Hooks**
Hooks defined trong skill frontmatter:

```markdown
---
name: code-reviewer
description: Auto-lint review of code changes
hooks:
 PreToolUse:
  - matcher: "Bash"
   hooks:
    - type: command
     command: "./scripts/validate-command.sh"
 PostToolUse:
  - matcher: "Edit|Write"
   hooks:
    - type: command
     command: "./scripts/run-linter.sh"
---

You are an expert code review agent...
```

### Pattern 3: **Smart Hook Dispatcher**
Thay vì trigger expensive validation mỗi bash command:

```bash
#!/bin/bash
# smart-hook-dispatcher.sh
INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command')

# Only run expensive validation for specific commands
case "$COMMAND" in
 *"terraform apply"*|*"kubectl apply"*|*"npm publish"*)
  ./scripts/expensive-validation.sh
  ;;
 *)
  exit 0
  ;;
esac
```

### Pattern 4: **Prompt Hook cho Nuanced Decisions**
Claude yes/no evaluation cho decisions phức tạp:

```json
{
 "hooks": {
  "PreToolUse": [{
   "matcher": "Bash",
   "hooks": [{
    "type": "prompt",
    "prompt": "Does this command affect production infrastructure? Answer only 'yes' or 'no'.",
    "deny-on": "yes"
   }]
  }]
 }
}
```

### Pattern 5: **Team-Based Builder/Validator**
Từ `disler/claude-code-hooks-mastery`:
- **Builder** subagent: Implement feature
- **Validator** subagent: PostToolUse hook triggers validation
- Exit 2 nếu validation fail → Builder retry

---

## ️ Common Pitfalls

### 1. **Wrong Exit Code** (trips up almost everyone)
- `exit 1` = hook error (Claude thinks hook failed)
- `exit 2` = block action
- `exit 0` = proceed

### 2. **Infinite Loops**
**Symptoms**: Claude keeps retrying, machine heats up.

**Stop hook causing actions** → Stop hook should only do **passive** things (log, notify, cleanup). NEVER write files or run commands Claude might respond to.

**Fix**: Check `stop_hook_active`:
```bash
#!/bin/bash
INPUT=$(cat)
if [ "$(echo "$INPUT" | jq -r '.stop_hook_active')" = "true" ]; then
 exit 0 # Allow stopping on subsequent invocations
fi
# ... your logic
```

### 3. **PostToolUse Modifying Files → Triggers Another PostToolUse**
Loop! Guard với specific matchers hoặc `if` field.

### 4. **Heavy Scripts trong Hot Paths**
PreToolUse/PostToolUse fire **frequently**. Heavy scripts slow everything.

**Fix**:
- Set `timeout` field (milliseconds)
- Cache results trong temp files
- Consider HTTP hook (async) cho heavy work

### 5. **No Caching**
Checking same thing repeatedly (vd "is this protected branch?"):
```bash
CACHE_FILE="/tmp/claude-branch-cache"
if [ -f "$CACHE_FILE" ] && [ $(($(date +%s) - $(stat -f %m "$CACHE_FILE"))) -lt 60 ]; then
 cat "$CACHE_FILE"
else
 git branch --show-current | tee "$CACHE_FILE"
fi
```

### 6. **PreToolUse Hook trong Non-Interactive Mode**
`PermissionRequest` hooks **không fire** với `-p`. Dùng `PreToolUse` thay thế.

### 7. **No Error Handling**
Hooks fail → messy transcript errors. Gracefully handle:
```bash
set -e
trap 'echo "Hook error: $?" >&2; exit 0' ERR
```

---

## Security Features

### PreToolUse `permissionDecision: "deny"`
**Cực mạnh**: Block tool ngay cả trong `bypassPermissions` mode hoặc `--dangerously-skip-permissions`.

```bash
#!/bin/bash
INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command')

if echo "$COMMAND" | grep -q "production"; then
 echo '{"permissionDecision": "deny", "reason": "Production commands require manual approval"}'
 exit 0
fi
exit 0
```

→ Enforce policy users **cannot bypass** bằng cách change permission mode.

---

## Testing Hooks

### Manual test
```bash
echo '{"tool_name":"Bash","tool_input":{"command":"ls"}}' | ./my-hook.sh
echo $? # Check exit code
```

### Debug tips
- **Toggle verbose mode**: `Ctrl+O` → see stdout/stderr of hooks
- **Check hook running**: `/hooks` command lists all configured
- **Test matchers**: case-sensitive! `Bash` ≠ `bash`
- **Check script executable**: `chmod +x ./my-hook.sh`

---

## Locations & Precedence

```
1. ~/.claude/settings.json       # User-global
2. .claude/settings.json        # Project-shared (commit git)
3. .claude/settings.local.json     # Project-personal (gitignore)
```

**Precedence**: Local > Project > User.

**Array settings merge** across scopes (hooks concatenate, không replace).

---

## Quick Install Order

### Week 1: Essential 3
- [ ] Auto-format on edit (PostToolUse)
- [ ] Block dangerous commands (PreToolUse)
- [ ] Desktop notification (Stop)

### Week 2: Quality Gates
- [ ] Auto-run tests (PostToolUse)
- [ ] Type check after edits (PostToolUse)
- [ ] Git context on start (SessionStart)

### Week 3: Advanced
- [ ] Smart hook dispatcher
- [ ] Protected path blocking
- [ ] Context usage monitoring (check-context hook)
- [ ] Session logging

### Month 2: Expert
- [ ] PreToolUse input modification
- [ ] Prompt hooks cho context-dependent decisions
- [ ] Skill-embedded hooks
- [ ] HTTP hooks cho Slack/external integrations

---

## Sources
- Anthropic Official: https://code.claude.com/docs/en/hooks-guide
- **Must-read**: `disler/claude-code-hooks-mastery` - comprehensive examples: https://github.com/disler/claude-code-hooks-mastery
- ClaudeLog: https://claudelog.com/mechanics/hooks/
- techsy.io (production-ready): https://techsy.io/en/blog/claude-code-hooks-guide
- Pixelmojo (12 events): https://www.pixelmojo.io/blogs/claude-code-hooks-production-quality-ci-cd-patterns
- Pasquale Pillitteri: https://pasqualepillitteri.it/en/news/657/claude-code-hooks-complete-guide
- DataCamp tutorial: https://www.datacamp.com/tutorial/claude-code-hooks
- SmartScope (Japanese, translated): https://smartscope.blog/en/generative-ai/claude/claude-code-hooks-guide/
