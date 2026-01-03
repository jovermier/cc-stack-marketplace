---
description: Automatically install relevant Claude Code plugins based on project context
---

# Auto Plugin Installation

Automatically installs Claude Code plugins from configured marketplaces based on detected project context.

## When to Use

This skill is invoked when:
- `/auto-skills` command is run

## Plugin Mapping

Based on project context, automatically install:

### Language-Specific Plugins

| Detected Language | Install Plugin |
|-------------------|----------------|
| Go | `cc-go@cc-stack-marketplace` |
| GraphQL | `cc-graphql@cc-stack-marketplace` |
| Next.js | `cc-nextjs@cc-stack-marketplace` |
| Playwright | `cc-playwright@cc-stack-marketplace` |

### Official Plugins

| Context | Install Plugin |
|---------|----------------|
| Any Node.js project | `typescript@claude-plugins-official` |
| Git repository | `git@claude-plugins-official` |
| Python project | `python@claude-plugins-official` |

## Installation Process

1. Check currently installed plugins
2. Compare with detected project needs
3. Install missing relevant plugins
4. Skip already installed plugins
5. Use `TMPDIR` workaround if needed

## Commands Used

```bash
# List installed plugins
claude plugin list

# Install from marketplace
TMPDIR=/home/coder/.claude/tmp claude plugin install <plugin>@<marketplace>
```

## Example Workflow

Given project context:
```json
{
  "languages": ["go"],
  "frameworks": ["graphql"],
  "testing": ["playwright"]
}
```

Actions:
1. Check if `cc-go` is installed → if not, install it
2. Check if `cc-graphql` is installed → if not, install it
3. Check if `cc-playwright` is installed → if not, install it

## Safety Rules

- Never remove existing plugins
- Always ask before installing if >3 plugins would be installed
- Respect user's `enabledPlugins` settings
- Log all installation actions
