# Agent Model Optimizer

Task-appropriate model routing for AI coding agents.

A **Cursor Agent Skill** that generates platform-specific configuration so subagents and delegated tasks use the right model tier — fast for exploration and shell work, balanced for implementation and tests, capable for review and debugging, premium for security and architecture.

**Supported tools:** Cursor · Claude Code · Codex · Windsurf

## What it does

- Maps **task types** to **capability tiers** (`fast` → `high-thinking`) via a single routing matrix
- Resolves tiers to platform-specific model IDs (Claude `haiku`/`sonnet`/`opus`, Cursor `composer-2.5-fast`, etc.)
- Generates **only the file set for your tool** — never installs configs you do not use
- Ships default agents: `explorer`, `reviewer`, `tester` (+ optional `security-auditor`, `architect`, `docs-writer`)
- Respects existing config: **merge** / **overwrite** / **skip** / **field-merge** (`model:` only)

## What it does not claim

Routing depends on your workflow, subagent call frequency, and task complexity. This skill helps you **match model capability to task type** — it does not guarantee specific cost savings or quality outcomes.

## Generated files (per tool)

| Tool | Output |
|------|--------|
| **Cursor** | `.cursor/rules/model-routing.mdc`, `.cursor/agents/*.md` |
| **Claude Code** | `.claude/agents/*.md`, `CLAUDE.md` (routing section) |
| **Codex** | `AGENTS.md` (agents section) |
| **Windsurf** | `.windsurfrules` (routing guidance) |

## Quick start

1. Copy this folder to `~/.cursor/skills/agent-model-optimizer/` (or clone the repo).
2. In Cursor, ask:
   > *"Use the agent-model-optimizer skill to set up model routing for this project."*
3. Confirm tool (auto-detected when possible), scope (project vs user), and profile:
   - `budget` — prefer lower tiers
   - `balanced` — default (recommended)
   - `quality` — prefer higher tiers
4. Choose conflict policy if existing config is found.
5. Review the routing summary table.

### Example prompt (Korean)

> agent-model-optimizer 스킬로 이 프로젝트 Cursor 모델 라우팅 설정해줘

## Default routing

| Agent | Task | Tier |
|-------|------|------|
| `explorer` | Codebase search | fast |
| `reviewer` | Code review | balanced-thinking |
| `tester` | Tests | balanced |
| `security-auditor` | Security audit | high-thinking |
| `architect` | Architecture / migration | high-thinking |
| `docs-writer` | Documentation | fast |

Built-in subagents (`explore`, `shell`, `bugbot` on Cursor) are documented in generated rules; their models are product-controlled.

## Repository layout

```
SKILL.md                    # Skill workflow (start here)
references/
  routing-matrix.md         # Task → tier (edit policy here)
  model-catalog.md          # Tier → model IDs (edit when models change)
  claude-code.md, cursor.md, codex.md, windsurf.md
templates/<tool>/           # Templates with {{TIER_*}} placeholders
DESIGN.md                   # Architecture and maintenance guide
```

## Customization

| Goal | Edit |
|------|------|
| Change which tier a task uses | `references/routing-matrix.md` |
| Update model IDs after a provider release | `references/model-catalog.md` |
| Change agent prompts or structure | `templates/<tool>/` |

Re-run the skill after editing the skill repo to regenerate project configs.

## Documentation

- [DESIGN.md](./DESIGN.md) — architecture, workflow, conflict policy
- [README.ko.md](./README.ko.md) — Korean overview

## License

TBD
