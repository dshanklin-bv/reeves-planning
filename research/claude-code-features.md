# Claude Code CLI — Features Enabling Autonomous Swarms

## What Claude Code Is

Claude Code is Anthropic's official CLI for Claude. It's the execution substrate for Reeves 3.0 — every slash command, ralph loop, and autonomous agent runs through it.

## Key Features for Reeves 3.0

### 1. Headless Mode (`claude -p`)
Run Claude without interactive terminal. Pipe in a prompt, get back results.

```bash
echo "Build the feature described in PRD.md" | claude -p
```

**Why it matters:** maxralphie spawns 5-8 headless Claude instances in parallel, each working on a different workstream. No human interaction needed.

### 2. Task Tool (Parallel Subagents)
Launch up to 10 specialized agents from within a session:

```
Task tool:
  subagent_type: "general-purpose"
  run_in_background: true
  prompt: "Build feature X..."
```

**Agent types available:**
- `general-purpose` — Full tool access, autonomous execution
- `Explore` — Fast codebase exploration
- `Plan` — Architecture planning
- `Bash` — Command execution
- `feature-dev:code-reviewer` — Code review
- `feature-dev:code-architect` — Architecture design

**Why it matters:** ralphie uses this to spawn 5-10 parallel build agents.

### 3. Hooks System
Shell commands that execute in response to events:

- **Pre-tool hooks** — Run before a tool executes
- **Post-tool hooks** — Run after a tool completes
- **Stop hooks** — Run when Claude tries to exit

**Why it matters:** Ralph Loop uses stop hooks to re-feed the same prompt, creating self-correcting iteration cycles.

### 4. Slash Commands (`.claude/commands/`)
Custom commands loaded from the project directory:

```
.claude/commands/
  mindreader.md    → /mindreader
  maxralph.md      → /maxralph
  ralphie.md       → /ralphie
```

**Why it matters:** The entire three-tier command hierarchy (/mindreader → /maxralphie → /ralphie) is implemented as slash commands.

### 5. MCP Server Integration
Connect to external services via Model Context Protocol:

**Currently connected:**
- `reeves` — 100+ tools for tasks, finance, people, graph, brain, etc.
- `pal` — 15 tools across GPT-5.2, Gemini, O3
- `tosh` — Apple Messages, contacts, photos
- `github` — Issues, PRs, code search
- `outlook` — Email, calendar
- `supabase` — Database operations
- `linear` — Project management
- `browsermcp` — Browser automation

**Why it matters:** mindreader queries 8+ MCP servers simultaneously to build a complete picture of the user's life.

### 6. CLAUDE.md Project Instructions
Repository-level instructions that persist across sessions:

```
/project-root/CLAUDE.md  → Project conventions, key files, architecture
~/.claude/CLAUDE.md       → Global user preferences, constraints
```

**Why it matters:** Every agent (ralph, maxralphie, mindreader) inherits project context automatically. No need to re-explain conventions.

### 7. Auto Memory (`.claude/memory/`)
Persistent memory files that survive across conversations:

```
.claude/projects/{path}/memory/MEMORY.md  → Auto-loaded into system prompt
.claude/projects/{path}/memory/*.md       → Topic-specific memory files
```

**Why it matters:** Patterns learned in one session inform the next. Debugging insights persist. This is cross-session memory for the development process itself.

### 8. Background Task Execution
Run agents in the background while the main session continues:

```
Task tool:
  run_in_background: true
  → Returns output_file path
  → Check with Read tool or tail
```

**Why it matters:** ralphie spawns all agents in background, then monitors progress. The main session stays responsive.

### 9. Context Compression
Automatic compression of prior messages as context limits approach:

**Why it matters:** Long sessions (like maxralphie's full pipeline) don't crash from context overflow. The system gracefully compresses older messages while preserving recent context.

### 10. Multi-Tool Parallel Calls
Call multiple tools simultaneously in a single message:

```
# These all execute in parallel:
- Read file A
- Read file B
- Grep for pattern C
- Query MCP tool D
```

**Why it matters:** mindreader's 8 parallel data gatherers, ralphie's multi-agent spawn, maxralphie's parallel PRD generation — all depend on parallel tool execution.

## The Ralph Loop Pattern

```bash
while :; do
  cat PROMPT.md | claude-code --continue
done
```

The same prompt is fed repeatedly. Claude sees its own previous work in files. Each iteration improves on the last. Stop hook intercepts exit attempts and re-feeds the prompt.

**Key properties:**
- Self-referential through file state (not output feedback)
- Deterministically bad in an undeterministic world (failures are predictable)
- Completion signaled by `<promise>` tag
- Max iterations prevent infinite loops

## Practical Limits

| Resource | Limit | Implication |
|----------|-------|-------------|
| Parallel subagents | 10 | ralphie caps at 10 concurrent tasks |
| Context window | 1M tokens | Fits entire codebase + data in one session |
| Background tasks | No hard limit | But diminishing returns past 10-15 |
| MCP connections | No hard limit | All servers stay connected |
| Session duration | No hard limit | Context compression handles long sessions |
