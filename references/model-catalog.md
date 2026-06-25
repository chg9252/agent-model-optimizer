# Model Catalog

Maps capability tiers to platform-specific model identifiers.  
**Update this file when providers ship or deprecate models.** Templates use placeholders; do not hardcode IDs in templates.

> Catalog version: 2026-06-24

## Placeholders

| Placeholder | Tier |
|-------------|------|
| `{{TIER_FAST}}` | `fast` |
| `{{TIER_BALANCED}}` | `balanced` |
| `{{TIER_BALANCED_THINKING}}` | `balanced-thinking` |
| `{{TIER_HIGH_THINKING}}` | `high-thinking` |

## Claude Code

Aliases or full model IDs accepted by Claude Code `model:` frontmatter.

| Tier | Model ID |
|------|----------|
| `fast` | `haiku` |
| `balanced` | `sonnet` |
| `balanced-thinking` | `sonnet` |
| `high-thinking` | `opus` |

Substitution map:

```
{{TIER_FAST}}              → haiku
{{TIER_BALANCED}}          → sonnet
{{TIER_BALANCED_THINKING}} → sonnet
{{TIER_HIGH_THINKING}}     → opus
```

## Cursor

Used in `.cursor/agents/*.md` frontmatter and Task `model` parameter when delegating.

| Tier | Model ID |
|------|----------|
| `fast` | `composer-2.5-fast` |
| `balanced` | `claude-4.6-sonnet-medium-thinking` |
| `balanced-thinking` | `claude-4.6-sonnet-medium-thinking` |
| `high-thinking` | `claude-opus-4-8-thinking-high` |

Substitution map:

```
{{TIER_FAST}}              → composer-2.5-fast
{{TIER_BALANCED}}          → claude-4.6-sonnet-medium-thinking
{{TIER_BALANCED_THINKING}} → claude-4.6-sonnet-medium-thinking
{{TIER_HIGH_THINKING}}     → claude-opus-4-8-thinking-high
```

Built-in subagents (`explore`, `shell`, `bugbot`, etc.) use product defaults — not overridden by this catalog.

## Codex

Used in `AGENTS.md` agent `model:` fields.

| Tier | Model ID |
|------|----------|
| `fast` | `gpt-5.5-medium` |
| `balanced` | `gpt-5.5-medium` |
| `balanced-thinking` | `gpt-5.3-codex` |
| `high-thinking` | `gpt-5.3-codex` |

Substitution map:

```
{{TIER_FAST}}              → gpt-5.5-medium
{{TIER_BALANCED}}          → gpt-5.5-medium
{{TIER_BALANCED_THINKING}} → gpt-5.3-codex
{{TIER_HIGH_THINKING}}     → gpt-5.3-codex
```

## Windsurf

Windsurf does not expose per-subagent model IDs in `.windsurfrules` the way Claude Code and Cursor do. The partial template uses **tier names and behavioral guidance** instead of model slugs.

| Tier | Windsurf expression |
|------|---------------------|
| `fast` | Prefer Cascade Fast / lightweight mode for exploration and shell tasks |
| `balanced` | Default coding model for implementation and tests |
| `balanced-thinking` | Standard model with explicit reasoning for review and debug |
| `high-thinking` | Most capable available model for security and architecture |

No `{{TIER_*}}` substitution in Windsurf partial — tier labels are written as human-readable guidance.

## Optional agents (all platforms)

| Agent | Tier key | Notes |
|-------|----------|-------|
| `security-auditor` | `high-thinking` | Respect profile floor in routing-matrix |
| `architect` | `high-thinking` | Same |
| `docs-writer` | `fast` (`quality` → `balanced`) | Resolve via profile first |
