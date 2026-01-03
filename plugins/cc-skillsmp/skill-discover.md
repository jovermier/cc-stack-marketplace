---
description: Discover skills using AI semantic search from SkillsMP.com marketplace
---

# SkillsMP AI Skill Discovery

Discover Claude skills using AI-powered semantic search. Finds skills based on meaning and intent, not just keywords.

## Usage

```
/skill-discover <natural-language-query>
```

## Examples

- `/skill-discover I need help with database migrations` - Find database migration skills
- `/skill-discover how to optimize react rendering` - Find React optimization skills
- `/skill-discover create rest api with authentication` - Find API and auth skills
- `/skill-discover fix memory leaks in go` - Find Go debugging skills

## How it works

1. Checks `SKILL_INSTALL_LOCATION` env var for target location (default: `workspace`)
2. Calls SkillsMP AI API: `GET /api/v1/skills/ai-search?q=<query>` with API key from `SKILLSMP_API_KEY` env var
3. Uses Cloudflare AI for semantic understanding of your query
4. Returns skills that match your intent, even if they don't contain exact keywords
5. Reviews each skill for relevance to the current project
6. Prompts you to select skills to install
7. Clones the skill's GitHub repo to the configured location

## Installation locations

The installation location is controlled by `SKILL_INSTALL_LOCATION` env var:

| Value | Location | Scope |
|-------|----------|-------|
| `workspace` (default) | `~/.claude/skills/` | Shared across all projects in workspace |
| `project` | `.claude/skills/` | Project-specific, shared with team |

Skills are automatically discovered by Claude Code when placed in these directories.

## API Details

**Endpoint**: `GET https://skillsmp.com/api/v1/skills/ai-search`

**Parameters**:
- `q` (required): Natural language search query

**Headers**:
- `Authorization: Bearer SKILLSMP_API_KEY`

## Example Implementation

When user runs `/skill-discover help with go database performance`:

```bash
# Fetch semantically matching skills
curl -X GET "https://skillsmp.com/api/v1/skills/ai-search?q=help+with+go+database+performance" \
  -H "Authorization: Bearer $SKILLSMP_API_KEY"
```

Review results, then install selected skill:
```bash
# Determine installation location
SKILL_DIR="${SKILL_INSTALL_LOCATION:-workspace}"
if [ "$SKILL_DIR" = "workspace" ]; then
  TARGET="$HOME/.claude/skills"
else
  TARGET=".claude/skills"
fi

# Install skill
mkdir -p "$TARGET"
git clone https://github.com/user/repo.git "$TARGET/skill-name"
```

## Keyword vs AI Search

| `/skill-add` | `/skill-discover` |
|--------------|-------------------|
| Keyword matching | Semantic understanding |
| `go testing` | `how to write better tests in go` |
| Faster results | Smarter results |
| Exact term matching | Intent-based matching |
