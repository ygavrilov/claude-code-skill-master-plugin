# Claude Code Skills Documentation

This file contains the complete reference documentation for creating Claude Code skills.

## Skill File Structure

Skills extend what Claude can do. Each skill is a directory containing:

```
skill-name/
├── SKILL.md           # Main instructions (required)
├── template.md        # Template for Claude to fill in (optional)
├── examples/          # Example output (optional)
│   └── sample.md
└── scripts/           # Scripts Claude can execute (optional)
    └── helper.py
```

## Skill Locations

| Location   | Path                                          | Applies to                     |
|:-----------|:----------------------------------------------|:-------------------------------|
| Enterprise | See managed settings                          | All users in your organization |
| Personal   | `~/.claude/skills/<skill-name>/SKILL.md`     | All your projects              |
| Project    | `.claude/skills/<skill-name>/SKILL.md`       | This project only              |
| Plugin     | `<plugin>/skills/<skill-name>/SKILL.md`      | Where plugin is enabled        |

## SKILL.md Structure

Every SKILL.md has two parts:

1. **YAML frontmatter** (between `---` markers) - Configuration
2. **Markdown content** - Instructions Claude follows

### Example SKILL.md

```yaml
---
name: explain-code
description: Explains code with visual diagrams and analogies. Use when explaining how code works.
allowed-tools: Read, Grep, Glob
context: fork
agent: Explore
disable-model-invocation: false
user-invocable: true
argument-hint: [filename]
---

When explaining code, always include:

1. **Start with an analogy**: Compare the code to something from everyday life
2. **Draw a diagram**: Use ASCII art to show the flow
3. **Walk through the code**: Explain step-by-step what happens
4. **Highlight a gotcha**: What's a common mistake?
```

## Frontmatter Fields

All fields are optional except where noted.

| Field                      | Required    | Description                                                                                                                                           |
|:---------------------------|:------------|:------------------------------------------------------------------------------------------------------------------------------------------------------|
| `name`                     | No          | Display name for the skill. If omitted, uses the directory name. Lowercase letters, numbers, and hyphens only (max 64 characters).                   |
| `description`              | Recommended | What the skill does and when to use it. Claude uses this to decide when to apply the skill. If omitted, uses the first paragraph of markdown content.|
| `argument-hint`            | No          | Hint shown during autocomplete to indicate expected arguments. Example: `[issue-number]` or `[filename] [format]`.                                   |
| `disable-model-invocation` | No          | Set to `true` to prevent Claude from automatically loading this skill. Use for workflows you want to trigger manually with `/name`. Default: `false`.|
| `user-invocable`           | No          | Set to `false` to hide from the `/` menu. Use for background knowledge users shouldn't invoke directly. Default: `true`.                             |
| `allowed-tools`            | No          | Tools Claude can use without asking permission when this skill is active.                                                                            |
| `model`                    | No          | Model to use when this skill is active.                                                                                                              |
| `context`                  | No          | Set to `fork` to run in a forked subagent context.                                                                                                   |
| `agent`                    | No          | Which subagent type to use when `context: fork` is set. Options: `Explore`, `Plan`, `general-purpose`, or custom agent name.                         |
| `hooks`                    | No          | Hooks scoped to this skill's lifecycle.                                                                                                              |

### Field Details

#### `allowed-tools`

List tools Claude can use without permission. Format:

- Tool name: `Read`, `Write`, `Edit`, `Grep`, `Glob`
- Tool with any arguments: `Bash(git *)`, `Bash(npm *)`
- Specific commands: `Bash(git status)`, `Bash(ls -la)`

Examples:
```yaml
allowed-tools: Read, Grep, Glob
```

```yaml
allowed-tools: Read, Write, Edit, Bash(git *), Bash(npm *)
```

#### `context: fork`

Runs skill in isolated subagent. The skill content becomes the subagent's prompt.

**Use when:**
- The skill is a self-contained task
- You want isolation from main conversation history
- The skill has explicit step-by-step instructions

**Don't use when:**
- Skill contains only guidelines/conventions (no actionable task)
- You want Claude to use skill knowledge in current conversation

#### `agent`

Specifies which subagent type to use when `context: fork` is set.

**Built-in agents:**
- `Explore`: Read-only codebase exploration
- `Plan`: Planning and design tasks
- `general-purpose`: Full tool access (default)

**Custom agents:**
- Reference any agent in `.claude/agents/`

#### `disable-model-invocation`

Controls whether Claude can automatically invoke the skill.

**Set to `true` for:**
- Skills with side effects (deploy, commit, send messages)
- Workflows you want to control timing of
- Actions that should only run when explicitly requested

**Leave `false` (default) for:**
- Reference knowledge Claude should use automatically
- Analysis skills Claude should apply when relevant

#### `user-invocable`

Controls whether skill appears in `/` command menu.

**Set to `false` for:**
- Background knowledge skills
- Skills that only make sense for Claude to invoke
- Internal/helper skills

## Dynamic Content and Substitutions

### String Substitutions

| Variable               | Description                                                                              |
|:-----------------------|:-----------------------------------------------------------------------------------------|
| `$ARGUMENTS`           | All arguments passed when invoking the skill                                            |
| `$ARGUMENTS[N]`        | Specific argument by 0-based index (e.g., `$ARGUMENTS[0]`)                              |
| `$N`                   | Shorthand for `$ARGUMENTS[N]` (e.g., `$0`, `$1`, `$2`)                                  |
| `${CLAUDE_SESSION_ID}` | The current session ID                                                                   |

**Example:**
```yaml
---
name: migrate-component
description: Migrate a component from one framework to another
---

Migrate the $0 component from $1 to $2.
Preserve all existing behavior and tests.
```

Usage: `/migrate-component SearchBar React Vue`

### Command Injection: !`command`

The `!`command`` syntax runs shell commands BEFORE sending skill content to Claude. Output replaces the placeholder.

**Example:**
```yaml
---
name: pr-summary
description: Summarize changes in a pull request
allowed-tools: Bash(gh *)
---

## Pull request context
- PR diff: !`gh pr diff`
- PR comments: !`gh pr view --comments`
- Changed files: !`gh pr diff --name-only`

## Your task
Summarize this pull request...
```

**How it works:**
1. Each `!`command`` executes immediately (preprocessing)
2. Output replaces the placeholder
3. Claude receives the fully-rendered prompt with actual data

This is NOT something Claude executes - it happens before Claude sees the content.

### File Imports: @filename

Reference supporting files so Claude knows what they contain:

```markdown
## Additional resources

- For complete API details, see @reference.md
- For usage examples, see @examples.md
```

## Skill Invocation Control

| Frontmatter                      | You can invoke | Claude can invoke | When loaded into context                                     |
|:---------------------------------|:---------------|:------------------|:------------------------------------------------------------|
| (default)                        | Yes            | Yes               | Description always in context, full skill loads when invoked |
| `disable-model-invocation: true` | Yes            | No                | Description not in context, full skill loads when you invoke |
| `user-invocable: false`          | No             | Yes               | Description always in context, full skill loads when invoked |

## Common Skill Patterns

### Reference Skills (Background Knowledge)

```yaml
---
name: api-conventions
description: API design patterns for this codebase
---

When writing API endpoints:
- Use RESTful naming conventions
- Return consistent error formats
- Include request validation
```

**Characteristics:**
- Default invocation settings (both user and Claude can invoke)
- Contains guidelines, not step-by-step tasks
- Runs inline (no `context: fork`)

### Task Skills (Explicit Actions)

```yaml
---
name: deploy
description: Deploy the application to production
context: fork
disable-model-invocation: true
---

Deploy the application:
1. Run the test suite
2. Build the application
3. Push to the deployment target
4. Verify deployment succeeded
```

**Characteristics:**
- `disable-model-invocation: true` (manual invocation only)
- Often uses `context: fork` for isolation
- Contains step-by-step instructions

### Argument-Based Skills

```yaml
---
name: fix-issue
description: Fix a GitHub issue
disable-model-invocation: true
argument-hint: [issue-number]
---

Fix GitHub issue $ARGUMENTS following our coding standards.

1. Read the issue description
2. Understand the requirements
3. Implement the fix
4. Write tests
5. Create a commit
```

### Dynamic Context Skills

```yaml
---
name: analyze-pr
description: Analyze pull request changes
allowed-tools: Bash(gh *)
---

## Current PR State
- Diff: !`gh pr diff`
- Status checks: !`gh pr checks`

Analyze the above and provide recommendations.
```

### Research Skills (Subagent)

```yaml
---
name: deep-research
description: Research a topic thoroughly
context: fork
agent: Explore
---

Research $ARGUMENTS thoroughly:

1. Find relevant files using Glob and Grep
2. Read and analyze the code
3. Summarize findings with specific file references
```

## Tool Permission Format

Tools specified in `allowed-tools` can be:

**Exact tool names:**
```yaml
allowed-tools: Read, Write, Edit, Grep, Glob
```

**Tool with wildcarded arguments:**
```yaml
allowed-tools: Bash(git *), Bash(npm *)
```

**Specific commands:**
```yaml
allowed-tools: Bash(git status), Bash(ls -la)
```

**Combined:**
```yaml
allowed-tools: Read, Grep, Glob, Bash(git *), Bash(python *)
```

## Supporting Files

Keep SKILL.md under 500 lines. Move detailed reference to separate files:

```
my-skill/
├── SKILL.md              # Overview and navigation
├── reference.md          # Detailed API docs
├── examples.md           # Usage examples
└── scripts/
    └── helper.py         # Utility script
```

Reference from SKILL.md:
```markdown
## Additional resources

- For complete API details, see @reference.md
- For usage examples, see @examples.md
```

## Extended Thinking

To enable extended thinking (thinking mode) in a skill, include the word "ultrathink" anywhere in your skill content.

## Best Practices

1. **Keep descriptions clear and keyword-rich** - Claude uses them to decide when to invoke
2. **Use `disable-model-invocation: true` for side effects** - Deploy, commit, send messages
3. **Use `context: fork` for isolated tasks** - Research, analysis, generation
4. **Provide `argument-hint` for clarity** - Shows users what to pass
5. **Use supporting files for large reference material** - Keep SKILL.md focused
6. **Test both invocation methods** - Manual (`/skill-name`) and automatic
7. **Grant minimal `allowed-tools`** - Only what the skill needs
8. **Use command injection sparingly** - Only for truly dynamic content
9. **Name skills with hyphens** - lowercase-with-hyphens (max 64 chars)
10. **Write task content for `context: fork`** - Skill must contain actionable instructions

## Troubleshooting

### Skill not triggering
- Check description includes keywords users would say
- Verify skill appears in "What skills are available?"
- Try invoking directly with `/skill-name`

### Skill triggers too often
- Make description more specific
- Add `disable-model-invocation: true`

### Claude doesn't see all skills
- Skill descriptions may exceed context budget (2% of context window, min 16K chars)
- Override with `SLASH_COMMAND_TOOL_CHAR_BUDGET` environment variable

## Additional Resources

- **Subagents**: Delegate tasks to specialized agents
- **Plugins**: Package and distribute skills
- **Hooks**: Automate workflows around tool events
- **Memory (CLAUDE.md)**: Persistent context across sessions
- **Permissions**: Control tool and skill access
