# Routing Matrix

Canonical task type → capability tier mapping. Platform-agnostic.  
When generating configs, apply a user **profile** bias (see below) before looking up model IDs in [model-catalog.md](model-catalog.md).

## Capability tiers

| Tier | Intent | Typical tasks |
|------|--------|---------------|
| `fast` | Low latency, low cost; mechanical or read-heavy work | File search, glob/grep, shell/CI, simple edits |
| `balanced` | Default for most coding work | Implementation, refactoring, test writing |
| `balanced-thinking` | Coding plus moderate reasoning | PR review, non-trivial debugging, API design |
| `high-thinking` | Deep reasoning; higher cost justified | Security audit, architecture, complex migrations |
| `inherit` | Match parent session model | Work tightly coupled to main conversation |

Tier order (low → high): `fast` < `balanced` < `balanced-thinking` < `high-thinking`

## Default agents

| Agent | Task type | Default tier | Notes |
|-------|-----------|--------------|-------|
| `explorer` | Codebase exploration | `fast` | Read/search only |
| `reviewer` | Code review | `balanced-thinking` | Judgment over speed |
| `tester` | Test authoring & fixing | `balanced` | Pattern-following work |

## Optional agents

Generate only when the user requests them.

| Agent | Task type | Default tier | Profile notes |
|-------|-----------|--------------|---------------|
| `security-auditor` | Security review | `high-thinking` | Never downgrade below `balanced-thinking` on `budget` |
| `architect` | Architecture / migration planning | `high-thinking` | Same floor as security |
| `docs-writer` | Docs / comments (non-critical) | `fast` | `quality` → `balanced` |

## Task type reference

| Task type | Default tier | Rationale |
|-----------|--------------|-----------|
| Codebase exploration | `fast` | Read/search dominant |
| Shell / CI / scripted ops | `fast` | Deterministic; failures visible in output |
| Routine implementation | `balanced` | Standard coding |
| Test authoring / fixing | `balanced` | Mostly pattern-following |
| Code review | `balanced-thinking` | Needs judgment |
| Debugging | `balanced-thinking` | Hypothesis-driven |
| Security review | `high-thinking` | False negatives are costly |
| Architecture / migration | `high-thinking` | Multi-file, long horizon |
| Docs / comments | `fast` | `quality` profile → `balanced` |

## Built-in delegation (Cursor / Claude Code)

These are product built-ins — tier is fixed by the tool. Parent rules should prefer them over `generalPurpose` when applicable.

| Built-in | Use for | Typical tier |
|----------|---------|--------------|
| `explore` / Explore | Quick codebase search | `fast` (product default) |
| `shell` | Terminal commands, git, CI | `fast` |
| `generalPurpose` | Multi-step work needing tools + judgment | `balanced` or parent model |
| `bugbot` / `security-review` | Specialized review subagents | product default |

## User profiles

Profiles shift tiers one step when two tiers are both reasonable.

| Profile | Rule |
|---------|------|
| `budget` | Prefer one tier **lower** (minimum `fast`) |
| `balanced` | Use defaults as-is |
| `quality` | Prefer one tier **higher** (maximum `high-thinking`) |

### Profile application

```
tier_order = [fast, balanced, balanced-thinking, high-thinking]

budget:   index = max(0, index - 1)
balanced: index unchanged
quality:  index = min(len - 1, index + 1)
```

### Profile exceptions

- `security-auditor` and `architect` on `budget`: floor at `balanced-thinking` (do not downgrade to `fast` or `balanced`).
- `inherit` is never adjusted by profile.

## Resolving tier → model

1. Look up agent or task type in tables above.
2. Apply profile bias.
3. Map tier to platform model via [model-catalog.md](model-catalog.md).
4. Substitute `{{TIER_*}}` placeholders in templates.
