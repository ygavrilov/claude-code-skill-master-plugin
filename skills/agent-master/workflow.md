# Agent Master — Workflow

## Process

### 1. Understand Requirements

Ask the user:

- **What should this agent do?** (One focused task — agents work best with a single specialty)
- **What tools does it need?** (Minimal — only what the task actually requires)
- **Should it have restricted permissions?** (Read-only? No writes? Specific permission mode?)
- **Should it persist memory across sessions?** (user / project / local scope?)
- **Should it use a specific model?** (sonnet / opus / haiku / inherit)
- **Should it preload any skills?** (Domain knowledge to inject at startup)

### 2. Determine Frontmatter

**Required:**
- `name`: lowercase-with-hyphens
- `description`: keyword-rich; include "Use proactively" for auto-delegation

**Tool Access (pick one):**
- `tools` — allowlist: only listed tools available
- `disallowedTools` — denylist: inherit all except these
- Omit both — inherit all from parent conversation

**Model:**
- `model: inherit` — same as main conversation (default)
- `model: sonnet` / `opus` / `haiku` — alias
- Full ID: `claude-sonnet-4-6`, `claude-opus-4-7`

**Permission Mode:**
- `permissionMode: default` — standard prompts
- `permissionMode: acceptEdits` — auto-accept file edits in working dir
- `permissionMode: auto` — background classifier reviews commands
- `permissionMode: bypassPermissions` — skip all prompts (use with caution)
- `permissionMode: plan` — read-only exploration

**Memory:**
- `memory: user` — `~/.claude/agent-memory/<name>/` (cross-project)
- `memory: project` — `.claude/agent-memory/<name>/` (shared via git) ← recommended default
- `memory: local` — `.claude/agent-memory-local/<name>/` (not in git)

**Other fields:**
- `maxTurns` — cap agentic turns
- `isolation: worktree` — isolated git worktree copy
- `background: true` — always background
- `color` — red, blue, green, yellow, purple, orange, pink, cyan
- `effort` — low / medium / high / xhigh / max
- `skills` — skill names to preload into context

### 3. Write the System Prompt

```
Brief role statement.

When invoked:
1. Step one
2. Step two

Key practices / checklist / constraints.

Output format.
```

- One-sentence role
- Explicit numbered workflow
- What to look for or produce
- Output format
- What agent CANNOT do (if restricted)
- If `memory` set: instruct agent to read before starting, write after finishing

### 4. Choose Location

- **Project** `.claude/agents/<name>.md` — shared via version control (default)
- **Personal** `~/.claude/agents/<name>.md` — all projects

### 5. Create the File

Single `.md` file — no directory needed. After writing, remind user: run `/agents` or restart session to load immediately.

### 6. Validate

- [ ] Single `.md` at correct path
- [ ] YAML frontmatter valid
- [ ] `name` lowercase-with-hyphens
- [ ] `description` clearly describes when to delegate
- [ ] Tools grant only necessary access
- [ ] System prompt has clear role + numbered workflow
- [ ] If `memory` set: system prompt instructs agent to use it
- [ ] If `hooks` set: scripts exist and are executable

---

## Editing Existing Agents

1. Read the existing `.md` file
2. Understand current frontmatter and system prompt
3. Ask what to change
4. Make targeted edits
5. Remind user to reload with `/agents` or restart session

---

@examples.md
