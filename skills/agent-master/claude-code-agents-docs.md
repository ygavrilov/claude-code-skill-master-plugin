# Claude Code Subagents — Reference Documentation

## File Structure

Subagent files use YAML frontmatter + Markdown system prompt:

```markdown
---
name: my-agent
description: What this agent does and when to use it
tools: Read, Grep, Glob
model: sonnet
---

You are a specialist agent. Your system prompt goes here.

When invoked:
1. Step one
2. Step two
```

**Locations:**
- Project: `.claude/agents/<name>.md` — shared via version control
- Personal: `~/.claude/agents/<name>.md` — available in all projects

Subagents load at session start. After creating a file, run `/agents` or restart to load immediately.

---

## Frontmatter Fields

### Required

| Field | Description |
|-------|-------------|
| `name` | Unique identifier. Lowercase letters and hyphens only. |
| `description` | When Claude should delegate to this agent. Keyword-rich. Include "Use proactively" to encourage auto-delegation. |

### Tool Access

| Field | Description |
|-------|-------------|
| `tools` | Allowlist — only these tools available. Omit to inherit all. |
| `disallowedTools` | Denylist — inherit all except these. Applied before `tools`. |

If both are set: `disallowedTools` applied first, then `tools` resolves against remainder. A tool in both is removed.

**Tool syntax examples:**
```yaml
tools: Read, Grep, Glob, Bash
tools: Read, Edit, Bash(git *), Bash(npm test)
disallowedTools: Write, Edit
```

**Agent spawning restriction** (only relevant when agent runs as main thread via `--agent`):
```yaml
tools: Agent(worker, researcher), Read, Bash   # only these subagent types
tools: Agent, Read, Bash                        # any subagent
# omit Agent entirely = cannot spawn subagents
```

### Model

| Value | Behavior |
|-------|----------|
| `inherit` | Same model as main conversation (default if omitted) |
| `sonnet` | claude-sonnet alias |
| `opus` | claude-opus alias |
| `haiku` | claude-haiku alias |
| Full ID | e.g. `claude-sonnet-4-6`, `claude-opus-4-7` |

Model resolution order (highest wins):
1. `CLAUDE_CODE_SUBAGENT_MODEL` env var
2. Per-invocation model parameter
3. Subagent `model` frontmatter
4. Main conversation model

### Permission Modes

| Mode | Behavior |
|------|----------|
| `default` | Standard permission checking with prompts |
| `acceptEdits` | Auto-accept file edits and common filesystem commands in working dir |
| `auto` | Background classifier reviews commands and protected-directory writes |
| `dontAsk` | Auto-deny permission prompts (explicitly allowed tools still work) |
| `bypassPermissions` | Skip all permission prompts — use with caution |
| `plan` | Read-only exploration mode |

**Parent override rules:**
- If parent uses `bypassPermissions` or `acceptEdits` → takes precedence, cannot be overridden
- If parent uses `auto` → subagent inherits `auto`, `permissionMode` in frontmatter is ignored

### Memory (Persistent Across Sessions)

| Scope | Location | Use when |
|-------|----------|----------|
| `user` | `~/.claude/agent-memory/<name>/` | Knowledge applicable across all projects |
| `project` | `.claude/agent-memory/<name>/` | Project-specific, shareable via version control |
| `local` | `.claude/agent-memory-local/<name>/` | Project-specific, not in version control |

When `memory` is set:
- System prompt gains instructions for reading/writing to memory directory
- First 200 lines or 25KB of `MEMORY.md` injected into context
- `Read`, `Write`, `Edit` tools auto-enabled for memory management

**Best practice:** Include in system prompt:
```
Update your agent memory as you discover codepaths, patterns, library
locations, and key architectural decisions.
```

### Other Fields

| Field | Type | Description |
|-------|------|-------------|
| `maxTurns` | integer | Maximum agentic turns before agent stops |
| `skills` | list | Skill names to preload into context at startup. Full skill content injected — not just made available |
| `mcpServers` | list | MCP servers for this agent. Inline definitions (scoped to agent) or string references (shared from session) |
| `hooks` | map | Lifecycle hooks scoped to this agent |
| `background` | bool | Always run as background task. Default: false |
| `effort` | string | `low`, `medium`, `high`, `xhigh`, `max` — overrides session effort level |
| `isolation` | string | `worktree` — run in isolated git worktree copy, auto-cleaned if no changes made |
| `color` | string | Display color: `red`, `blue`, `green`, `yellow`, `purple`, `orange`, `pink`, `cyan` |
| `initialPrompt` | string | Auto-submitted as first user turn when agent runs as main session (via `--agent`). Commands and skills processed. Prepended to user-provided prompt. |

---

## MCP Servers

```yaml
mcpServers:
  # Inline definition — scoped to this agent only
  - playwright:
      type: stdio
      command: npx
      args: ["-y", "@playwright/mcp@latest"]
  # String reference — reuses already-configured server
  - github
```

Inline definitions use same schema as `.mcp.json` (stdio, http, sse, ws). They connect when agent starts, disconnect when it finishes. Use inline to keep server tools out of the main conversation context.

---

## Hooks in Frontmatter

Hooks run while the agent is active (subagent or main session via `--agent`):

```yaml
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
  Stop:
    - hooks:
        - type: command
          command: "./scripts/cleanup.sh"
```

`Stop` hooks in frontmatter are automatically converted to `SubagentStop` events when running as subagent.

**Hook input:** JSON via stdin. Extract with jq:
```bash
INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // empty')
```

**Exit codes:**
- `0` — allow
- `2` — block (stderr returned to Claude as error message)

---

## Project-Level Hooks for Subagent Events

In `settings.json`, respond to subagent lifecycle:

```json
{
  "hooks": {
    "SubagentStart": [
      {
        "matcher": "db-agent",
        "hooks": [{ "type": "command", "command": "./scripts/setup-db.sh" }]
      }
    ],
    "SubagentStop": [
      {
        "hooks": [{ "type": "command", "command": "./scripts/cleanup.sh" }]
      }
    ]
  }
}
```

---

## Skills Preloading

```yaml
skills:
  - api-conventions
  - error-handling-patterns
```

Full skill content injected at startup. Subagents don't inherit skills from parent — must list explicitly. Cannot preload skills with `disable-model-invocation: true`.

---

## Invocation Patterns

| Pattern | Example | Effect |
|---------|---------|--------|
| Natural language | "Use the code-reviewer agent" | Claude decides whether to delegate |
| @-mention | `@"code-reviewer (agent)"` | Guarantees that agent runs |
| Session-wide | `claude --agent code-reviewer` | Entire session uses agent's system prompt |
| Settings default | `{ "agent": "code-reviewer" }` in `.claude/settings.json` | Default for every session in project |

**@-mention syntax:**
- Local: `@agent-<name>`
- Plugin: `@agent-<plugin-name>:<agent-name>`

---

## Foreground vs Background

| | Foreground | Background |
|-|------------|------------|
| Execution | Blocks until complete | Concurrent with main conversation |
| Permission prompts | Passed through to user | Pre-approved before launch, then auto-denied |
| AskUserQuestion | Passed through | Fails silently, agent continues |

Force background: ask Claude "run this in the background" or press `Ctrl+B`.

Disable all background: `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS=1`.

---

## Disable Specific Agents

In `settings.json`:
```json
{
  "permissions": {
    "deny": ["Agent(Explore)", "Agent(my-custom-agent)"]
  }
}
```

Or via CLI: `claude --disallowedTools "Agent(Explore)"`

---

## Fork Subagents (Experimental)

Enable: `CLAUDE_CODE_FORK_SUBAGENT=1`

A fork inherits the full conversation context instead of starting fresh — same system prompt, tools, model, message history. Use when a named agent would need too much background re-explanation.

| | Fork | Named Subagent |
|-|------|----------------|
| Context | Full conversation history | Fresh — only what you pass |
| System prompt | Same as main session | From agent's definition file |
| Model | Same as main session | From agent's `model` field |
| Prompt cache | Shared with main session | Separate cache |

Start a fork manually: `/fork draft unit tests for the parser changes so far`

Limitations: fork cannot spawn further forks.

---

## Example Agents

### Code Reviewer (Read-Only)
```yaml
---
name: code-reviewer
description: Expert code review specialist. Proactively reviews code for quality, security, and maintainability. Use immediately after writing or modifying code.
tools: Read, Grep, Glob, Bash
model: inherit
---

You are a senior code reviewer ensuring high standards of code quality and security.

When invoked:
1. Run git diff to see recent changes
2. Focus on modified files
3. Begin review immediately

Review checklist:
- Code clarity and readability
- Well-named functions and variables
- No duplicated code
- Proper error handling
- No exposed secrets or API keys
- Input validation
- Test coverage
- Performance considerations

Provide feedback by priority: Critical → Warnings → Suggestions.
Include specific fix examples.
```

### Debugger (Read + Write)
```yaml
---
name: debugger
description: Debugging specialist for errors, test failures, and unexpected behavior. Use proactively when encountering any issues.
tools: Read, Edit, Bash, Grep, Glob
---

You are an expert debugger specializing in root cause analysis.

When invoked:
1. Capture error message and stack trace
2. Identify reproduction steps
3. Isolate the failure location
4. Implement minimal fix
5. Verify solution works

Focus on fixing the underlying issue, not the symptoms.
```

### Data Scientist
```yaml
---
name: data-scientist
description: Data analysis expert for SQL queries, BigQuery operations, and data insights. Use proactively for data analysis tasks and queries.
tools: Bash, Read, Write
model: sonnet
---

You are a data scientist specializing in SQL and BigQuery analysis.

When invoked:
1. Understand the data analysis requirement
2. Write efficient SQL queries
3. Use BigQuery command line tools (bq) when appropriate
4. Analyze and summarize results
5. Present findings clearly

Write optimized queries. Include comments for complex logic. Provide data-driven recommendations.
```

### DB Reader (Hook-Validated)
```yaml
---
name: db-reader
description: Execute read-only database queries. Use when analyzing data or generating reports.
tools: Bash
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-readonly-query.sh"
---

You are a database analyst with read-only access. Execute SELECT queries only.
You cannot modify data. If asked to INSERT, UPDATE, DELETE, or modify schema, explain you only have read access.
```

Validation script (`./scripts/validate-readonly-query.sh`):
```bash
#!/bin/bash
INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // empty')
if [ -z "$COMMAND" ]; then exit 0; fi
if echo "$COMMAND" | grep -iE '\b(INSERT|UPDATE|DELETE|DROP|CREATE|ALTER|TRUNCATE|REPLACE|MERGE)\b' > /dev/null; then
  echo "Blocked: Write operations not allowed. Use SELECT queries only." >&2
  exit 2
fi
exit 0
```

Make executable: `chmod +x ./scripts/validate-readonly-query.sh`
