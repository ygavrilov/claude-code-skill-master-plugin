# Skill Master — Workflow

## Process

### 1. Understand Requirements

Ask the user:

- **What should this skill do?**
- **When should it be invoked?** (Automatically by Claude, manually by user, or both?)
- **Should it run in isolation?** (Does it need `context: fork`?)
- **What tools does it need?**
- **Does it need arguments?**

### 2. Determine Frontmatter

**Required:**
- `name`: lowercase-with-hyphens, max 64 chars
- `description`: keyword-rich, drives automatic invocation

**Invocation Control:**
- `disable-model-invocation: true` — user-triggered only (deploy, commit, side effects)
- `user-invocable: false` — Claude-only background knowledge

**Execution Context:**
- `context: fork` — isolated subagent
- `agent: Explore|Plan|general-purpose` — which subagent (requires `context: fork`)

**Tools:**
- `allowed-tools` — e.g. `Read, Grep, Glob, Bash(git *)`

**UX:**
- `argument-hint` — e.g. `[issue-number]`, `[filename] [format]`

### 3. Write Skill Content

**Reference Skills** — guidelines/conventions, no tasks, inline, no `context: fork`

**Task Skills** — numbered steps, often `context: fork` + `disable-model-invocation: true`

**Dynamic Skills** — `$ARGUMENTS`, `$0`/`$1` for input; `` !`pwd` `` for shell injection; `@filename` for supporting files

### 4. Choose Location

- **Personal** `~/.claude/skills/<name>/SKILL.md` — all projects
- **Project** `.claude/skills/<name>/SKILL.md` — current project only

Default to personal unless user specifies otherwise.

### 5. Create Supporting Files

Move content to dedicated files (see principles.md for structure rules):
- Workflow/process → `workflow.md`
- Examples/patterns → `examples.md`
- Reference docs → `reference.md` or domain-specific named files
- Scripts → `scripts/`

### 6. Validate

- [ ] `name` is lowercase-with-hyphens
- [ ] `description` is clear and keyword-rich
- [ ] `allowed-tools` correct format
- [ ] `context: fork` has actionable content (not just guidelines)
- [ ] Arguments use `$ARGUMENTS` or `$N`
- [ ] Supporting files referenced with `@filename`

---

## Tips

1. Keyword-rich descriptions — use terms users say naturally
2. `context: fork` for isolation — research, analysis, generation
3. Minimal tools — only what the skill needs
4. Test both `/skill-name` and natural language invocation
5. `` !`pwd` `` sparingly — only for truly dynamic content
6. Argument hints — help users know what to pass
7. Clear names — the name becomes the `/command`

---

## Editing Existing Skills

1. Read the existing SKILL.md
2. Understand current structure and frontmatter
3. Ask what to change
4. Make targeted edits with Edit tool
5. Explain what changed and why

---

@examples.md
