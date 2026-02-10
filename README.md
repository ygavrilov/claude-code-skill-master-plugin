# Claude Code Skill Master Plugin

A Claude Code plugin that adds the **Skill Master** skill — a guided assistant for creating and editing Claude Code skills.

## What it does

Skill Master walks you through building properly structured Claude Code skills. It asks about your skill's purpose, invocation method, tools, and arguments, then generates a valid `SKILL.md` with correct frontmatter and content.

It covers:

- Creating new skills (reference, task, dynamic, or research patterns)
- Editing existing skills
- Choosing the right frontmatter fields (`context`, `allowed-tools`, `disable-model-invocation`, etc.)
- Placing skills in personal (`~/.claude/skills/`) or project (`.claude/skills/`) directories
- Splitting large skills into supporting files

## Install

Add to your Claude Code plugins:

```
claude plugin add /path/to/claude-code-skill-master-plugin
```

## Usage

Once installed, invoke the skill:

```
/skill-master
```

Or describe what you need in natural language — Claude will invoke it automatically when you mention creating or editing a skill.

## Structure

```
plugin.json                          # Plugin metadata
skills/
  skill-master/
    SKILL.md                         # Skill definition and instructions
    claude-code-skills-docs.md       # Reference documentation
```

## Author

Yevgeniy Gavrilov
