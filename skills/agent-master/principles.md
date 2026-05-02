# Agent Structure Principles

## One Agent, One Specialty

Focused agents outperform generalists. Each agent should excel at exactly one task. If the scope feels broad, split it.

## Agent Files Are Single .md Files

Unlike skills, agents are a single `.md` file — no directory. Location:
- Project: `.claude/agents/<name>.md`
- Personal: `~/.claude/agents/<name>.md`

## Description Drives Delegation

Claude reads `description` to decide when to delegate. Make it:
- Keyword-rich (use terms users say naturally)
- Explicit about trigger conditions
- Include "Use proactively" to enable auto-delegation without being asked

## Minimal Tool Access

Grant only what the task requires. Prefer `tools` allowlist over `disallowedTools` denylist when the needed set is small. Never add tools "just in case."

## Memory Scope Default

`memory: project` is the recommended default — project-specific knowledge, shareable via git. Use `memory: user` only for cross-project knowledge. Use `memory: local` when knowledge must not be committed.

When memory is enabled, the system prompt must instruct the agent to:
1. Read memory before starting work
2. Write new discoveries after finishing

## permissionMode: bypassPermissions

Only for fully trusted, well-scoped agents. Never use as a shortcut to avoid thinking about permissions.

## isolation: worktree

Use when the agent makes file changes that should be reviewed before merging into the working tree.

## Skills Preloading

List skills under `skills:` when the agent needs domain knowledge at startup. Skills are fully injected — not just made available. Agents do not inherit skills from the parent conversation.
