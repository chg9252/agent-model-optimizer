# Windsurf Platform Guide

## Output paths

| Template | Target |
|----------|--------|
| `templates/windsurf/windsurfrules.partial` | `.windsurfrules` (merge) |

## Limitations

Windsurf does **not** support per-subagent model IDs in `.windsurfrules` the way Claude Code (`.claude/agents/`) or Cursor (`.cursor/agents/`) do. This skill generates:

- **Behavioral routing rules** — which task types should use lightweight vs capable models
- **Tier labels** (`fast`, `balanced`, etc.) as guidance for the user to match in Windsurf's model picker

Set honest expectations in the post-generation summary: user may need to manually select Cascade Fast vs Pro for certain workflows.

## Partial merge: .windsurfrules

Replace or append delimited block:

```markdown
<!-- agent-model-optimizer:start -->
...
<!-- agent-model-optimizer:end -->
```

No `{{TIER_*}}` substitution — tier names appear as readable guidance (see [model-catalog.md](model-catalog.md) Windsurf section).

## Generation checklist

- [ ] Merge `windsurfrules.partial` into `.windsurfrules`
- [ ] Summarize manual model selection hints per tier
- [ ] Do not write `.cursor/` or `.claude/` files in the same run
