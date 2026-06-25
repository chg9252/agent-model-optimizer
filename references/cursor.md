# Cursor Platform Guide

## Output paths

| Template | Target path |
|----------|-------------|
| `templates/cursor/rules/model-routing.mdc` | `.cursor/rules/model-routing.mdc` |
| `templates/cursor/agents/*.md` | `.cursor/agents/<name>.md` |

Default scope: **project** (`.cursor/`). User-level: `~/.cursor/agents/` when user requests global agents.

## Two routing layers

### Layer 1: Built-in subagents (product-controlled)

| Subagent | Delegate when | Model |
|----------|---------------|-------|
| `explore` | Codebase search, file discovery, keyword lookup | Product default (fast) |
| `shell` | Terminal, git, npm, CI commands | Product default |
| `bugbot` | User asks for bug-focused code review | Product default |
| `security-review` | User asks for security review | Product default |
| `generalPurpose` | Multi-step tasks needing varied tools | Avoid for simple search/shell |

Parent agent follows `.cursor/rules/model-routing.mdc` to pick the right built-in instead of defaulting to `generalPurpose`.

### Layer 2: Custom agents (user-controlled)

`.cursor/agents/<name>.md`:

```markdown
---
name: reviewer
description: Expert code review. Use after code changes or before PR.
model: claude-4.6-sonnet-medium-thinking
---

System prompt body.
```

Include `model:` from catalog substitution. When invoking via Task tool, pass the same `model` slug for consistency.

## model-routing.mdc

- `alwaysApply: true` so parent agent always sees delegation rules.
- Content: when to use built-ins vs custom agents; never spawn `generalPurpose` for grep-only work.

## Partial files

Cursor outputs are **not** partial — write `model-routing.mdc` and agent files directly. On conflict:

| File | Default policy |
|------|----------------|
| New agent file | `merge` (write) |
| Existing agent | Ask: `overwrite`, `skip`, or `field-merge` (update `model:` only) |
| `model-routing.mdc` exists | Ask: `overwrite` or `skip` |

## Generation checklist

- [ ] Substitute `{{TIER_*}}` in agent templates
- [ ] Write `.cursor/rules/model-routing.mdc`
- [ ] Write `.cursor/agents/explorer.md`, `reviewer.md`, `tester.md`
- [ ] Optional agents if user requested
- [ ] Do not write Claude Code or Windsurf files in the same run
