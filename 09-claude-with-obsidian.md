# Topic 9: Best Practices Using Claude Với Obsidian

> **Nguồn**: GitHub repos (iansinnott, MarkusPfundstein, AgriciDaniel), dev.to, Obsidian forum, awesomeclaude.ai

---

## 🎯 Tổng Quan

Obsidian = markdown vault local → Claude có thể access theo **3 cách chính**.

### 3 Options Integration (từ awesomeclaude.ai)

| Option | Pros | Cons | Ai nên dùng |
|--------|------|------|-------------|
| **1. Filesystem MCP** | Đơn giản nhất, không plugin Obsidian | Không dùng Obsidian search features | Beginners |
| **2. Obsidian Local REST API + mcp-obsidian** | Full vault access, search, patch content | Cần 2 components setup | Power users |
| **3. obsidian-claude-code-mcp plugin** | Native plugin, WebSocket | Port conflicts nếu multiple vaults | Claude Code users |

---

## 🥇 Option 1: Filesystem MCP (Đơn giản nhất)

### Concept
Obsidian vault = folder markdown files → Claude Filesystem MCP access trực tiếp, không cần Obsidian running.

### Setup
```bash
# Install Filesystem MCP
claude mcp add filesystem \
  -s user \
  npx @modelcontextprotocol/server-filesystem \
  /path/to/your/vault
```

### Pros
- ✅ Zero Obsidian plugins needed
- ✅ Works offline
- ✅ Claude có thể work khi Obsidian closed

### Cons
- ❌ Không search bằng Obsidian's indexing
- ❌ Không hiểu [[wiki links]] natively (chỉ text)
- ❌ Không trigger Obsidian plugins/workflows

### Best for
- Dev notes, code docs trong vault
- Khi bạn primarily dùng Claude Code

---

## 🥈 Option 2: Local REST API + mcp-obsidian (Recommended Full Integration)

### Architecture
```
Claude Desktop/Code
      ↓ MCP Protocol
   mcp-obsidian (Python server)
      ↓ HTTP REST API
Obsidian (phải đang chạy)
   └── Local REST API plugin
```

### Setup Steps

#### 1. Install Obsidian Plugin
- Settings → Community plugins → Browse
- Search **"Local REST API"** → Install & Enable
- Copy API key từ plugin settings

#### 2. Install mcp-obsidian
**Repo**: https://github.com/MarkusPfundstein/mcp-obsidian

```bash
# Option A: qua uvx (recommended)
claude mcp add-json obsidian '{
  "command": "uvx",
  "args": ["mcp-obsidian"],
  "env": {
    "OBSIDIAN_API_KEY": "your_api_key_here",
    "OBSIDIAN_HOST": "127.0.0.1",
    "OBSIDIAN_PORT": "27124"
  }
}' --scope user
```

#### 3. Restart Claude

#### 4. Test
```
> Create a test note called "Integration Test" với content "Hello from Claude"
> List files in my Obsidian vault
```

### 8 Tools Available
| Tool | Chức năng |
|------|-----------|
| `list_files_in_vault` | List root directory |
| `list_files_in_dir` | List specific folder |
| `get_file_contents` | Read file |
| `search` | Search across vault |
| `patch_content` | Insert content relative to heading/block/frontmatter |
| `append_content` | Append to existing/new file |
| `delete_file` | Delete file/folder |
| (plus) | More utilities |

---

## 🥉 Option 3: obsidian-claude-code-mcp Plugin

### Concept
**Repo**: https://github.com/iansinnott/obsidian-claude-code-mcp

Plugin native cho Obsidian - MCP server chạy **trong** Obsidian, dùng WebSocket.

### Setup
1. Install plugin trong Obsidian (Community plugins → Browse → "Claude Code MCP")
2. Enable plugin
3. Default port: **22360** (tránh conflict với dev services)
4. Claude Code auto-discovers và connects

### Multi-vault Support
Mỗi vault cần port riêng. Plugin detect conflicts và guide config.

### Cho Claude Desktop
Claude Desktop không support HTTP transport trực tiếp. Dùng `mcp-remote` bridge:
```json
{
  "mcpServers": {
    "obsidian": {
      "command": "npx",
      "args": ["mcp-remote", "http://localhost:22360/sse"],
      "env": {}
    }
  }
}
```

---

## 🚀 Advanced: AgriciDaniel/claude-obsidian (Karpathy's LLM Wiki Pattern)

**Repo**: https://github.com/AgriciDaniel/claude-obsidian

### Unique concept
"Knowledge engine" thay vì "chat interface":
- Claude **creates**, **organizes**, **maintains**, **evolves** notes **autonomously**
- Persistent, compounding wiki vault
- Based on Andrej Karpathy's LLM Wiki pattern

### Commands
- `/wiki` - Enter wiki mode
- `/save` - Save current learnings to vault
- `/autoresearch` - Autonomous research workflow

### Pre-configured
- `setup-vault.sh` configures graph.json (filter + colors)
- `app.json` excludes plugin dirs
- `appearance.json` enables custom CSS
- Bases support (Obsidian v1.9.10+) thay thế Dataview

---

## 💡 Best Practice Workflows

### Workflow 1: Claude as Note Organizer
**Initial prompt** (tạo Project trong Claude Desktop):
```
Use the Obsidian MCP tools as much as possible to help me organize
and enhance my knowledge base. Always look for opportunities to
create meaningful connections between notes and maintain consistent structure.
```

**Example tasks**:
- "Reorganize notes about React into a logical hierarchy"
- "Add backlinks between related notes"
- "Convert raw meeting notes into structured format"
- "Generate weekly digest from this week's notes"

### Workflow 2: Research → Obsidian
```
Research [topic] using web search.
Create a comprehensive note "/Research/topic.md" với:
- Summary (TL;DR)
- Key findings
- Sources (linked)
- Related notes (use [[]] syntax)
- Tags (#research, #<topic>)
```

### Workflow 3: Daily Notes Automation
```
Create today's daily note at Daily/YYYY-MM-DD.md với template:
## Today's Focus
## Meetings
## Learnings
## Tomorrow

Then scan recent notes for anything I should review today.
```

### Workflow 4: Convert Chat → Vault
Transfer học từ Claude conversation vào vault:
```
I want you to dump this conversation about [topic] into my Obsidian
vault as a comprehensive study guide. Path: Learning/<topic>.md.
Use proper markdown formatting, code blocks, and [[links]] to related topics.
```

### Workflow 5: Dev Journal + Claude Code
CLAUDE.md trong project:
```markdown
# Project rules
When finishing significant work:
1. Update dev journal at ~/Obsidian/Dev/YYYY-MM-DD.md
2. Log: what was built, decisions made, problems solved
3. Link to relevant project notes
```

---

## 🎨 Useful Prompts Cho Obsidian Users

### "Ask my vault"
```
Search my Obsidian vault for notes about [topic] and summarize key points.
```

### "Build knowledge graph"
```
Analyze my vault and create a new note "/Meta/knowledge-map.md" with:
- Main topics I've been studying
- Connections between them (visual with [[links]])
- Gaps where I have minimal notes
```

### "Clean up structure"
```
Review my vault folder structure.
Suggest improvements for:
- Moving misfiled notes
- Renaming unclear titles
- Adding missing tags
- Creating MOCs (Maps of Content) for large topics
```

### "Enhance with connections"
```
Read note "[[ProjectX-notes]]".
Find 5 other notes in my vault that should link to this.
Update those notes với appropriate [[ProjectX-notes]] references.
```

---

## 🔒 Security Considerations

### ⚠️ Quan trọng
1. **API key sensitive** - don't share publicly
2. **Use separate test vault** đầu tiên trước khi connect main vault
3. **Regularly audit** files Claude created/modified
4. **Be cautious** sharing config files (có thể chứa paths/keys)
5. **Local only**: REST API port phải firewall, không expose public

### File Access Control
```json
{
  "permissions": {
    "deny": [
      "Read(/path/to/vault/.obsidian/workspace.json)",
      "Read(/path/to/vault/Private/**)"
    ]
  }
}
```
Chặn Claude access private folders.

---

## 🚨 Common Issues & Solutions

### Issue 1: "uv/uvx location not found"
**Fix**: Run `which uvx` và paste full path trong config:
```json
{
  "command": "/Users/you/.local/bin/uvx"
}
```

### Issue 2: "Unrecognised subcommand"
**Fix**: Argument order sai. Pattern đúng:
```
uv run mcp-obsidian  (not uv mcp-obsidian run)
```

### Issue 3: Claude không detect MCP tools
- Check Obsidian Local REST API plugin **enabled**
- Check Obsidian **đang chạy** khi requests
- Restart Claude after config changes
- Look for tool icon trong Claude interface

### Issue 4: Port conflict (multi-vault)
obsidian-claude-code-mcp tự detect → config different ports cho mỗi vault.

### Issue 5: Protocol version mismatch
obsidian-claude-code-mcp dùng **"HTTP with SSE" (2024-11-05)** - legacy protocol.
Newer "Streamable HTTP" (2025-03-26) chưa support bởi most MCP clients.

---

## 🎯 Path Best Practices

### Dùng relative paths từ vault root
```
❌ /Users/you/Documents/Obsidian/Vault/Work/note.md
✅ Work/Projects/ProjectA/notes.md
```

### Instruct Claude specifically
```
Create note at path: Research/AI/claude-workflows.md
(Use vault-relative path, not absolute)
```

---

## 📚 Sources & Repos

| Repo | Purpose |
|------|---------|
| [MarkusPfundstein/mcp-obsidian](https://github.com/MarkusPfundstein/mcp-obsidian) | Python MCP server via REST API |
| [iansinnott/obsidian-claude-code-mcp](https://github.com/iansinnott/obsidian-claude-code-mcp) | Native Obsidian plugin |
| [AgriciDaniel/claude-obsidian](https://github.com/AgriciDaniel/claude-obsidian) | LLM Wiki pattern, `/wiki` commands |
| [coddingtonbear/obsidian-local-rest-api](https://github.com/coddingtonbear/obsidian-local-rest-api) | Required Obsidian plugin |
| [jacksteamdev/obsidian-mcp-tools](https://github.com/jacksteamdev/obsidian-mcp-tools) | MCP Tools Obsidian plugin |

### Articles
- Obsidian Forum: https://forum.obsidian.md/t/automate-note-generation-in-obsidian-with-claude-desktop-and-mcp-servers/99542
- dev.to guide: https://dev.to/sroy8091/connect-claude-ai-with-obsidian-a-game-changer-for-knowledge-management-25o2
- QED42 blog: https://www.qed42.com/insights/supercharge-your-knowledge-management---integrating-obsidian-mcp-with-claude
- awesomeclaude.ai: https://awesomeclaude.ai/how-to/use-obsidian-with-claude
