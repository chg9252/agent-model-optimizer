---
name: agent-model-optimizer
description: >-
  Generates task-appropriate model routing configuration for AI coding tools
  (Cursor, Claude Code, Codex, Windsurf). Maps subagent and task types to model
  tiers and writes platform-specific config files. Use when the user mentions
  model optimization, subagent setup, model routing, token efficiency, or
  configuring agents per task type.
disable-model-invocation: true
---

# Agent Model Optimizer

Generate platform-specific model routing so subagents use the right capability tier per task.

**Respond to the user in their language.** Generated config files (frontmatter, model IDs, descriptions) stay in English.

Do not claim guaranteed cost savings or quality outcomes. Recommend routing; outcomes depend on workflow.

---

## When to use

- User wants subagent or task-level model routing
- User asks to reduce unnecessary spend on lightweight delegated work
- User sets up a new project with Cursor, Claude Code, Codex, or Windsurf
- User wants explorer / reviewer / tester agents with appropriate models

---

## Workflow overview

```
Detect → Profile → Resolve → Conflict policy → Generate → Summarize
```

Read [routing-matrix.md](references/routing-matrix.md) and [model-catalog.md](references/model-catalog.md) before generating. Read the platform guide for the selected tool.

---

## Step 1: Detect

### 1.1 Tool detection

| Signal | Tool |
|--------|------|
| `.cursor/` or user says Cursor | `cursor` |
| `CLAUDE.md`, `.claude/`, or user says Claude Code | `claude-code` |
| `AGENTS.md` with agent sections, Codex CLI context | `codex` |
| `.windsurfrules` or user says Windsurf | `windsurf` |
| Multiple or none | Ask: Cursor / Claude Code / Codex / Windsurf |

Generate **one tool only** per run.

### 1.2 Scope

| Scope | Claude Code | Cursor | Codex | Windsurf |
|-------|-------------|--------|-------|----------|
| Project (default) | `.claude/agents/` | `.cursor/agents/`, `.cursor/rules/` | `AGENTS.md` | `.windsurfrules` |
| User | `~/.claude/agents/` | `~/.cursor/agents/` | N/A | N/A |

Ask only if unclear. Default: project scope.

### 1.3 Existing config scan

List paths that would be written and flag conflicts:

**cursor:** `.cursor/rules/model-routing.mdc`, `.cursor/agents/{explorer,reviewer,tester,...}.md`

**claude-code:** `.claude/agents/*.md`, `CLAUDE.md` (partial merge)

**codex:** `AGENTS.md` (partial merge)

**windsurf:** `.windsurfrules` (partial merge)

Also check for existing `<!-- agent-model-optimizer:start -->` delimiter blocks.

---

## Step 2: Profile

Ask only if not inferable from conversation.

### 2.1 Tier bias

| Profile | Effect |
|---------|--------|
| `budget` | One tier lower (min `fast`) |
| `balanced` | Default matrix (recommended) |
| `quality` | One tier higher (max `high-thinking`) |

Apply rules from [routing-matrix.md](references/routing-matrix.md#user-profiles). Respect floors for `security-auditor` and `architect` on `budget`.

### 2.2 Optional agents

Default set: `explorer`, `reviewer`, `tester`.

Offer optionally: `security-auditor`, `architect`, `docs-writer`. Generate only what the user confirms.

For **codex** and **windsurf**, optional agents appear only inside the partial merge (codex) or as tier rows (windsurf) — skip unused agent sections when merging codex partial.

---

## Step 3: Resolve

1. For each agent to generate, look up default tier in [routing-matrix.md](references/routing-matrix.md).
2. Apply profile bias.
3. Map tier → model ID via [model-catalog.md](references/model-catalog.md) for the selected tool.
4. Read platform guide: [claude-code.md](references/claude-code.md) | [cursor.md](references/cursor.md) | [codex.md](references/codex.md) | [windsurf.md](references/windsurf.md).

### Placeholder substitution

Replace in template files before writing:

| Placeholder | Resolve from catalog |
|-------------|---------------------|
| `{{TIER_FAST}}` | `fast` row for tool |
| `{{TIER_BALANCED}}` | `balanced` row |
| `{{TIER_BALANCED_THINKING}}` | `balanced-thinking` row |
| `{{TIER_HIGH_THINKING}}` | `high-thinking` row |

Windsurf partial: no placeholder substitution (tier names as guidance only).

### Template source paths

| Tool | Templates |
|------|-----------|
| `cursor` | `templates/cursor/` |
| `claude-code` | `templates/claude-code/` |
| `codex` | `templates/codex/` |
| `windsurf` | `templates/windsurf/` |

Skill root: directory containing this `SKILL.md` (the `agent-model-optimizer` folder).

---

## Step 4: Conflict policy

If conflicts exist, ask unless the user already said "overwrite all" or similar.

| Policy | Behavior |
|--------|----------|
| `merge` (default) | Write new files; merge partials via delimiter block |
| `overwrite` | Replace entire file |
| `skip` | Leave existing file unchanged |
| `field-merge` | Update only `model:` in existing agent files |

**Never** silently overwrite customized files.

### Partial merge rules

Delimiter:

```markdown
<!-- agent-model-optimizer:start -->
...
<!-- agent-model-optimizer:end -->
```

| Target | Action |
|--------|--------|
| `CLAUDE.md` | Insert or replace delimited block; add `## Model routing` context if new file |
| `AGENTS.md` | Insert or replace delimited block under `## Agents` |
| `.windsurfrules` | Append or replace delimited block |

On re-run with `merge`, replace only content between delimiters.

For codex partial: omit `###` sections for agents the user did not request (keep default three unless optional agents confirmed).

---

## Step 5: Generate

### cursor

1. Write `templates/cursor/rules/model-routing.mdc` → `.cursor/rules/model-routing.mdc`
2. For each selected agent, write `templates/cursor/agents/<name>.md` → `.cursor/agents/<name>.md` with placeholders substituted

### claude-code

1. For each selected agent, write `templates/claude-code/agents/<name>.md` → `.claude/agents/<name>.md`
2. Merge `templates/claude-code/CLAUDE.md.partial` into `CLAUDE.md`

### codex

1. Build content from `templates/codex/AGENTS.md.partial` (only requested agents)
2. Merge into `AGENTS.md`

### windsurf

1. Merge `templates/windsurf/windsurfrules.partial` into `.windsurfrules`

Create parent directories if missing.

---

## Step 6: Summarize

Present in the user's language:

1. **Routing table** — agent/task | tier | resolved model ID (or Windsurf guidance)
2. **Files written** — paths and merge/overwrite/skip actions
3. **Built-in notes** (Cursor / Claude Code) — when to use `explore`, `shell`, etc.
4. **How to adjust** — edit tiers in `routing-matrix.md` or model IDs in `model-catalog.md` in the skill repo, then re-run

### Example summary (English)

```
Model routing configured for Cursor (project, balanced).

| Agent / task   | Tier              | Model                             |
|----------------|-------------------|-----------------------------------|
| explorer       | fast              | composer-2.5-fast                 |
| reviewer       | balanced-thinking | claude-4.6-sonnet-medium-thinking |
| tester         | balanced          | claude-4.6-sonnet-medium-thinking |
| explore (built-in) | product default | use for quick search          |
| shell (built-in)   | product default | use for terminal work         |

Files: .cursor/rules/model-routing.mdc, .cursor/agents/{explorer,reviewer,tester}.md
```

---

## Tool output matrix

| Tool | Files generated |
|------|-----------------|
| Cursor | `.cursor/rules/model-routing.mdc`, `.cursor/agents/*.md` |
| Claude Code | `.claude/agents/*.md`, `CLAUDE.md` (section) |
| Codex | `AGENTS.md` (section) |
| Windsurf | `.windsurfrules` (section) |

---

## Additional resources

- [routing-matrix.md](references/routing-matrix.md) — task → tier
- [model-catalog.md](references/model-catalog.md) — tier → model IDs
- [claude-code.md](references/claude-code.md)
- [cursor.md](references/cursor.md)
- [codex.md](references/codex.md)
- [windsurf.md](references/windsurf.md)
- [DESIGN.md](DESIGN.md) — full architecture

---

## Maintenance (for skill authors)

| Change | Edit |
|--------|------|
| New model from provider | `references/model-catalog.md` only |
| New task type | `references/routing-matrix.md` + optional template agent |
| Tool format change | `references/<tool>.md` + affected templates |
