# Skill Examples

## Manual-Only Task

```yaml
---
name: deploy
description: Deploy the application to production
disable-model-invocation: true
context: fork
allowed-tools: Bash(git *), Bash(npm *), Bash(ssh *)
---

Deploy to production:
1. Run test suite
2. Build application
3. Push to deployment target
```

## Automatic Reference

```yaml
---
name: coding-standards
description: Coding standards and conventions for this project
allowed-tools: Read
---
Follow these conventions:
    - Use TypeScript strict mode
    - Write tests for all public APIs
    - Document complex logic
```

## Argument-Based

```yaml
---
name: fix-issue
description: Fix a GitHub issue by number
disable-model-invocation: true
argument-hint: [issue-number]
allowed-tools: Bash(gh *), Read, Write, Edit
---

Fix GitHub issue #$ARGUMENTS:

1. Fetch issue details: `gh issue view $ARGUMENTS`
2. Understand requirements
3. Implement fix
4. Write tests
5. Create commit
```

## Research Subagent

```yaml
---
name: research-pattern
description: Research a code pattern across the codebase
context: fork
agent: Explore
---

Research $ARGUMENTS thoroughly:

1. Find relevant files using Glob and Grep
2. Read and analyze implementations
3. Identify common patterns and variations
4. Summarize findings with file references
```
