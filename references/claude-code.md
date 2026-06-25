# Claude Code Platform Guide

## Output paths

| Template | Target path | Scope |
|----------|-------------|-------|
| `templates/claude-code/agents/*.md` | `.claude/agents/<name>.md` | Project (default) or `~/.claude/agents/` (user) |
| `templates/claude-code/CLAUDE.md.partial` | `CLAUDE.md` (merge) | Project root |

## Subagent file format

```markdown
---
name: explorer
description: When to delegate (be specific; include trigger terms)
model: haiku
tools: Read, Glob, Grep
---

System prompt body.
```

### Frontmatter fields

| Field | Required | Notes |
|-------|----------|-------|
| `name` | Yes | Lowercase, hyphens |
| `description` | Yes | Drives auto-delegation |
| `model` | Recommended | From catalog substitution; `inherit` to match parent |
| `tools` | Optional | Restrict for read-only agents |

## Model resolution order (Claude Code)

1. `CLAUDE_CODE_SUBAGENT_MODEL` env var
2. Per-invocation `model` parameter
3. Subagent `model:` frontmatter
4. Parent conversation model

## Built-in Explore agent

Claude Code's built-in Explore already uses a fast model. Custom `explorer` is for team-specific prompts and tool restrictions. Do not duplicate unless the user wants shared project-level behavior.

## Partial merge: CLAUDE.md

1. If `CLAUDE.md` missing → create with partial content (no delimiter needed on first write).
2. If exists → insert or replace block between:
   ```markdown
   <!-- agent-model-optimizer:start -->
   ...
   <!-- agent-model-optimizer:end -->
   ```
3. Default conflict policy: `merge` (replace only the delimited block on re-run).

## Generation checklist

- [ ] Substitute all `{{TIER_*}}` from [model-catalog.md](model-catalog.md)
- [ ] Write agents to `.claude/agents/`
- [ ] Merge `CLAUDE.md.partial` into project `CLAUDE.md`
- [ ] Never write Cursor or Windsurf files in the same run
