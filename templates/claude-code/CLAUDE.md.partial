<!-- agent-model-optimizer:start -->

## Model routing

Subagents in `.claude/agents/` are tiered by task type. Prefer delegating instead of doing everything in the main session.

| Task | Agent | Tier |
|------|-------|------|
| Codebase search | `explorer` | fast |
| Code review | `reviewer` | balanced-thinking |
| Tests | `tester` | balanced |
| Security | `security-auditor` (if installed) | high-thinking |
| Architecture | `architect` (if installed) | high-thinking |

Built-in Explore is fine for quick search; use custom `explorer` when the team needs shared exploration rules.

<!-- agent-model-optimizer:end -->
