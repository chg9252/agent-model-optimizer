# Codex Platform Guide

## Output paths

| Template | Target |
|----------|--------|
| `templates/codex/AGENTS.md.partial` | `AGENTS.md` (project root, merge) |

Fallback: if user prefers Codex-only `instructions.md`, append equivalent agent block there using the same delimiter convention.

## AGENTS.md agent format

```markdown
## reviewer

description: Expert code review. Use before commits or PRs.
tools: read, bash
model: gpt-5.3-codex

You are a senior code reviewer...
```

Or YAML-style frontmatter under `###` headings — match existing project `AGENTS.md` style when merging.

## Partial merge: AGENTS.md

1. Detect existing `AGENTS.md` style (heading level, field format).
2. Insert or replace delimited block:
   ```markdown
   <!-- agent-model-optimizer:start -->
   ...
   <!-- agent-model-optimizer:end -->
   ```
3. Per-agent merge: if agent heading exists and policy is `field-merge`, update `model:` only.

## Generation checklist

- [ ] Substitute `{{TIER_*}}` from catalog
- [ ] Merge partial into `AGENTS.md`
- [ ] Include default three agents (+ optional if requested)
- [ ] Do not write `.cursor/` or `.claude/` files in the same run
