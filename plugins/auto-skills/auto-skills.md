---
description: Automatically detect project needs and install relevant skills from SkillsMP
---

# Auto Skills

Automatically detect your project's technology stack and install relevant skills from SkillsMP.

## Usage

```
/auto-skills
```

## What It Does

1. **Analyzes your project** to detect:
   - Programming languages
   - Frameworks and libraries
   - Testing frameworks
   - Package managers

2. **Fetches relevant skills** from SkillsMP:
   - Uses AI semantic search based on detected tech
   - Reviews top results
   - Installs up to 5 relevant skills to configured location

## Configuration

### Environment Variables

```bash
# Set API key for SkillsMP (required)
export SKILLSMP_API_KEY=sk_live_your_key

# Where to install skills (default: workspace)
export SKILL_INSTALL_LOCATION=workspace  # or "project"

# Max skills to install per run (default: 5)
export AUTO_SKILL_MAX=5

# Disable skill installation if needed
export AUTO_SKILL_ENABLED=false
```

### Project Settings

Add to `.claude/settings.json`:

```json
{
  "autoSkills": {
    "skipSkills": [],
    "preferredSkills": []
  }
}
```

## Example

```
/auto-skills
```

Output:
```
🔄 Auto Skills Analysis

Detected project:
  • Languages: Go, TypeScript
  • Frameworks: GraphQL, Next.js
  • Testing: Playwright

Skills to install (from SkillsMP):
  → go-performance (⭐ 234)
  → graphql-schema-design (⭐ 156)
  → nextjs-optimization (⭐ 203)

Install location: ~/.claude/skills/ (workspace)

Install 3 skills? [Y/n]
```

## Skill Installation Location

The `SKILL_INSTALL_LOCATION` env variable controls where skills are installed:

| Value | Location | Scope |
|-------|----------|-------|
| `workspace` (default) | `~/.claude/skills/` | Shared across all projects |
| `project` | `.claude/skills/` | Project-specific |

## Safety

- Always shows what will be installed before acting
- Never removes existing skills
- Skips already installed items
- Requires API key for SkillsMP integration
