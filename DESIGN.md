# Agent Model Optimizer — Design Document

> **Status:** Implemented v1  
> **Last updated:** 2026-06-24  
> **Language policy:** Repository internals in English; respond to users in their language at runtime.

---

## 1. Overview

### 1.1 Purpose

Agent Model Optimizer is a **Cursor Agent Skill** (and distributable template pack) that helps users configure **task-appropriate model routing** for AI coding tools. It generates platform-specific configuration files so that parent agents and subagents use the right model tier for each job — avoiding the common default of running every delegated task on the most capable (and most expensive) model.

### 1.2 Problem

Most users either:

- Run all subagent work on one premium model, or
- Leave subagents on `inherit` / default, which often resolves to the parent session's model.

Both patterns waste tokens on tasks that do not need maximum reasoning depth (exploration, shell commands, simple refactors) while under-provisioning genuinely hard tasks (security audits, complex migrations) when users blindly downgrade everything.

### 1.3 Solution

A single **routing matrix** (task type → capability tier) translated into each tool's native configuration format:

| Tool | Primary outputs |
|------|-----------------|
| Claude Code | `.claude/agents/*.md`, `CLAUDE.md` guidance |
| Cursor | `.cursor/rules/model-routing.mdc`, `.cursor/agents/*.md` |
| Codex | `AGENTS.md` or `instructions.md` |
| Windsurf | `.windsurfrules` |

### 1.4 Positioning (messaging)

**Use:**

- "Route each task type to an appropriate model tier."
- "Match model capability to task complexity."
- "Reduce unnecessary spend on lightweight subagent work."
- "Keep premium models for tasks that benefit from deeper reasoning."

**Avoid:**

- Guaranteed savings multipliers ("5× cheaper", "save 80%").
- Quality guarantees ("no quality loss").
- Time-sensitive model advice baked into templates without a catalog layer.

Savings and quality trade-offs **depend on workflow, codebase size, and how often subagents are invoked**. The skill recommends routing; it does not promise outcomes.

---

## 2. Design Principles

1. **Single source of truth** — Task → tier mapping lives in one file; platform files are projections.
2. **Progressive disclosure** — `SKILL.md` stays short; details live in `references/`.
3. **Generate one tool at a time** — Never install configs for tools the user does not use.
4. **Infer before asking** — Detect tool and existing config from the workspace; ask only when ambiguous.
5. **Separate tiers from model IDs** — Tiers are stable; model slugs live in a catalog that can be updated independently.
6. **Respect existing config** — Merge / overwrite / skip policies; never silently destroy user customizations.
7. **English internals, localized UX** — Repo and generated config in English; user-facing summaries in the user's language.

---

## 3. Repository Structure

```
agent-model-optimizer/
├── SKILL.md                         # Skill entry point (workflow only, <500 lines)
├── README.md                        # GitHub overview (English)
├── README.ko.md                     # Optional Korean overview
├── DESIGN.md                        # This document
│
├── references/
│   ├── routing-matrix.md            # Task type → capability tier (canonical)
│   ├── model-catalog.md             # Tier → platform-specific model IDs
│   ├── claude-code.md               # How to express tiers in Claude Code
│   ├── cursor.md                    # How to express tiers in Cursor
│   ├── codex.md                     # How to express tiers in Codex
│   └── windsurf.md                  # How to express tiers in Windsurf
│
└── templates/
    ├── claude-code/
    │   ├── agents/
    │   │   ├── explorer.md
    │   │   ├── reviewer.md
    │   │   └── tester.md
    │   └── CLAUDE.md.partial          # Snippet to merge into CLAUDE.md
    ├── cursor/
    │   ├── rules/
    │   │   └── model-routing.mdc
    │   └── agents/
    │       ├── explorer.md
    │       ├── reviewer.md
    │       └── tester.md
    ├── codex/
    │   └── AGENTS.md.partial
    └── windsurf/
        └── windsurfrules.partial
```

### 3.1 File responsibilities

| File | Role | Updated when |
|------|------|--------------|
| `SKILL.md` | Orchestration workflow | Workflow changes |
| `routing-matrix.md` | Task taxonomy and tier rules | New task types or tier policy |
| `model-catalog.md` | Concrete model slugs per platform | Provider releases new models |
| `references/<tool>.md` | Platform mapping notes | Tool config format changes |
| `templates/<tool>/` | Placeholder-based output skeletons | Template structure changes |

---

## 4. Core Components

### 4.1 Routing matrix (`references/routing-matrix.md`)

Canonical mapping from **task types** to **capability tiers**. Platform-agnostic.

#### Capability tiers

| Tier | Intent | Typical tasks |
|------|--------|---------------|
| `fast` | Low latency, low cost; mechanical or read-heavy work | File search, glob/grep exploration, shell/CI execution, simple edits |
| `balanced` | Default for most coding work | Implementation, refactoring, code review, test writing |
| `balanced-thinking` | Coding plus moderate reasoning | PR review, debugging non-trivial bugs, API design |
| `high-thinking` | Deep reasoning; higher cost justified | Security audit, architecture decisions, complex migrations |
| `inherit` | Match parent session model | Tasks tightly coupled to main conversation context |

#### Default task → tier mapping

| Task type | Tier | Rationale |
|-----------|------|-----------|
| Codebase exploration | `fast` | Read/search dominant; rarely needs premium reasoning |
| Shell / CI / scripted ops | `fast` | Deterministic steps; failures are visible in output |
| Routine implementation | `balanced` | Standard coding; quality matters, depth usually moderate |
| Test authoring / fixing | `balanced` | Pattern-following; occasional edge cases |
| Code review | `balanced-thinking` | Needs judgment; full premium often overkill |
| Debugging | `balanced-thinking` | Hypothesis-driven; depth scales with bug complexity |
| Security review | `high-thinking` | False negatives are costly |
| Architecture / migration planning | `high-thinking` | Multi-file reasoning, long horizon |
| Docs / comments (non-critical) | `fast` or `balanced` | User profile selects |

#### User profiles (tier bias)

Profiles adjust tier selection within the matrix — they do not replace it.

| Profile | Behavior |
|---------|----------|
| `budget` | Prefer one tier lower when two tiers are reasonable (e.g. review → `balanced` instead of `balanced-thinking`) |
| `balanced` | Use default matrix as-is |
| `quality` | Prefer one tier higher when trade-off is ambiguous |

### 4.2 Model catalog (`references/model-catalog.md`)

Maps each tier to **platform-specific model identifiers**. Templates use placeholders; the skill substitutes values at generation time.

#### Placeholder convention

```
{{TIER_FAST}}
{{TIER_BALANCED}}
{{TIER_BALANCED_THINKING}}
{{TIER_HIGH_THINKING}}
```

#### Example catalog shape (illustrative — update as providers ship new models)

| Tier | Claude Code | Cursor (Task / agents) | Codex | Windsurf |
|------|-------------|------------------------|-------|----------|
| `fast` | `haiku` | `composer-2.5-fast` | `gpt-5.5-medium` | *(tool-specific)* |
| `balanced` | `sonnet` | `claude-4.6-sonnet-medium-thinking` | `gpt-5.5-medium` | *(tool-specific)* |
| `balanced-thinking` | `sonnet` | `claude-4.6-sonnet-medium-thinking` | `gpt-5.3-codex` | *(tool-specific)* |
| `high-thinking` | `opus` | `claude-opus-4-8-thinking-high` | `gpt-5.3-codex` | *(tool-specific)* |

> **Maintenance rule:** When a model is deprecated, update **only** `model-catalog.md` and verify templates still render valid IDs.

### 4.3 Default agent catalog

Three starter subagents ship in every tool template set. Users may add more during skill execution.

| Agent | Task type | Default tier | Tools emphasis |
|-------|-----------|--------------|----------------|
| `explorer` | Codebase exploration | `fast` | Read-only / search |
| `reviewer` | Code review | `balanced-thinking` | Read, diff-aware |
| `tester` | Test authoring & fixing | `balanced` | Read, write tests |

Optional extensions (offered during workflow, not generated by default):

- `security-auditor` → `high-thinking`
- `architect` → `high-thinking`
- `docs-writer` → `fast` or `balanced`

---

## 5. Skill Workflow (`SKILL.md`)

`disable-model-invocation: true` — this skill runs on explicit user request, not ambiently.

### 5.1 Workflow steps

```
┌─────────────────────────────────────────────────────────────┐
│ 1. DETECT                                                   │
│    - Tool: infer from .cursor/, CLAUDE.md, .windsurfrules,  │
│      codex config; confirm if ambiguous                     │
│    - Scope: project vs user-level paths                       │
│    - Existing config: list conflicting files                  │
└──────────────────────────┬──────────────────────────────────┘
                           ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. PROFILE (ask only if unknown)                            │
│    - budget | balanced | quality                              │
│    - Optional: extra agents beyond default three              │
└──────────────────────────┬──────────────────────────────────┘
                           ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. RESOLVE                                                  │
│    - Read routing-matrix.md + apply profile bias              │
│    - Read model-catalog.md for selected tool                │
│    - Read references/<tool>.md for format rules             │
└──────────────────────────┬──────────────────────────────────┘
                           ▼
┌─────────────────────────────────────────────────────────────┐
│ 4. CONFLICT POLICY (ask if conflicts exist)                 │
│    - merge | overwrite | skip per file or globally          │
└──────────────────────────┬──────────────────────────────────┘
                           ▼
┌─────────────────────────────────────────────────────────────┐
│ 5. GENERATE                                                 │
│    - Substitute placeholders in templates/<tool>/           │
│    - Write only selected tool's file set                      │
│    - For .partial files: merge into existing target         │
└──────────────────────────┬──────────────────────────────────┘
                           ▼
┌─────────────────────────────────────────────────────────────┐
│ 6. SUMMARIZE (user's language)                              │
│    - Routing table: agent/task → tier → model ID            │
│    - Files written and merge actions taken                  │
│    - How to adjust later (edit matrix vs catalog)           │
└─────────────────────────────────────────────────────────────┘
```

### 5.2 Detection signals

| Signal | Likely tool |
|--------|-------------|
| `.cursor/` present | Cursor |
| `CLAUDE.md` or `.claude/` | Claude Code |
| `AGENTS.md` with agent frontmatter, Codex CLI context | Codex |
| `.windsurfrules` | Windsurf |
| None / multiple | Ask user |

### 5.3 Conflict policy

| Policy | When to use |
|--------|-------------|
| `merge` | Default for `.partial` snippets and new agent files that do not exist |
| `overwrite` | User explicitly wants a clean slate; backs up are recommended |
| `skip` | File exists and user has customizations to preserve |
| `field-merge` | Update only `model:` (or equivalent) in existing agent definitions |

**Never** silently overwrite without detection + user choice (or explicit "overwrite all" instruction).

### 5.4 Localization instruction (in SKILL.md)

```markdown
Respond to the user in their language. If the user writes in Korean,
summaries and explanations are in Korean; generated config files remain
in English (model IDs, frontmatter, descriptions).
```

---

## 6. Platform-Specific Design

### 6.1 Claude Code

**Outputs:**

- `.claude/agents/<name>.md` — subagent files with `model:` frontmatter
- `CLAUDE.md` — optional section via `CLAUDE.md.partial` merge

**Tier expression:**

```yaml
---
name: explorer
description: Fast codebase exploration. Read and search only.
model: {{TIER_FAST}}
tools: Read, Glob, Grep
---
```

**Notes:**

- Built-in `Explore` agent already uses a fast model; custom `explorer` complements or replaces team-specific exploration behavior.
- `inherit` is valid when subagent should match parent session.

**Reference:** `references/claude-code.md`

---

### 6.2 Cursor

**Outputs:**

- `.cursor/rules/model-routing.mdc` — parent agent behavior: when to delegate, which built-in or custom subagent, tier guidance
- `.cursor/agents/<name>.md` — custom subagents for team-shared routing

**Two layers (document clearly in `references/cursor.md`):**

| Layer | Configurable by user? | Role |
|-------|----------------------|------|
| Built-in subagents (`explore`, `shell`, `bugbot`, `generalPurpose`, …) | Limited — model often fixed by product | Parent rule tells *when* to use each |
| Custom `.cursor/agents/*.md` | Full — frontmatter + prompt | Team-defined agents with explicit tiers |

**`model-routing.mdc` purpose:**

- Instruct parent agent to pick `explore` for search, `shell` for commands, custom `reviewer` for reviews, etc.
- Document tier intent so parent does not spawn `generalPurpose` for grep tasks.

**Reference:** `references/cursor.md`

---

### 6.3 Codex

**Outputs:**

- `AGENTS.md` (preferred cross-tool format) or tool-specific `instructions.md`

**Tier expression:** Agent section with `model:` field per AGENTS.md conventions.

**Reference:** `references/codex.md`

---

### 6.4 Windsurf

**Outputs:**

- `.windsurfrules` via `windsurfrules.partial` merge

**Constraint:** Subagent model routing may be less granular than Claude Code / Cursor. `references/windsurf.md` documents what can and cannot be configured; skill sets honest expectations.

**Reference:** `references/windsurf.md`

---

## 7. Template System

### 7.1 Placeholder substitution

Templates contain `{{PLACEHOLDER}}` tokens. At generation time the skill:

1. Resolves tier from routing matrix + user profile.
2. Looks up model ID in catalog for selected tool.
3. Replaces placeholders in template files.
4. Writes to target paths per tool conventions.

### 7.2 Partial files

Files named `*.partial` are **not** written verbatim. They are merged:

| Target | Merge strategy |
|--------|----------------|
| `CLAUDE.md` | Append `## Model routing` section if missing; skip if section exists (unless overwrite) |
| `AGENTS.md` | Insert `## Agents` block or merge per-agent |
| `.windsurfrules` | Append routing section with clear delimiter comments |

Delimiter example:

```markdown
<!-- agent-model-optimizer:start -->
... generated content ...
<!-- agent-model-optimizer:end -->
```

Enables idempotent re-runs and surgical updates.

---

## 8. SKILL.md Outline (implementation target)

```markdown
---
name: agent-model-optimizer
description: >
  Generates task-appropriate model routing configuration for AI coding tools
  (Cursor, Claude Code, Codex, Windsurf). Maps subagent/task types to model
  tiers and writes platform-specific config files. Use when the user mentions
  model optimization, subagent setup, model routing, token efficiency, or
  configuring agents per task type.
disable-model-invocation: true
---

# Agent Model Optimizer

## When to use
## Step 1: Detect tool, scope, and existing config
## Step 2: Profile (budget | balanced | quality)
## Step 3: Resolve routing (read routing-matrix, model-catalog, references)
## Step 4: Conflict policy
## Step 5: Generate from templates
## Step 6: Summarize routing table (user's language)

## Additional resources
- [routing-matrix.md](references/routing-matrix.md)
- [model-catalog.md](references/model-catalog.md)
- Platform guides: claude-code, cursor, codex, windsurf
```

---

## 9. Implementation Phases

Follow this order (matches recommended authoring sequence):

| Phase | Deliverable | Done when |
|-------|-------------|-----------|
| **0** | `DESIGN.md` (this doc), `README.md` | Messaging and structure agreed |
| **1** | `references/routing-matrix.md`, `references/model-catalog.md` | Done |
| **2** | `references/*.md` (four platform guides) | Done |
| **3** | `templates/` for all four tools | Done |
| **4** | `SKILL.md` | Done |
| **5** | `README.ko.md` (optional) | Done |
| **6** | Dogfood on a real project | Pending |

---

## 10. Distribution

### 10.1 Install paths

| Audience | Location |
|----------|----------|
| Personal skill | `~/.cursor/skills/agent-model-optimizer/` |
| Project skill | `.cursor/skills/agent-model-optimizer/` (if repo copied in) |
| GitHub clone | Users clone repo; copy skill folder or symlink |

### 10.2 Versioning

- Tag releases when `model-catalog.md` changes (e.g. `v1.0.0`, `v1.1.0-catalog`).
- Changelog entry: which tiers/platforms were affected.

---

## 11. Maintenance Playbook

| Event | Action |
|-------|--------|
| New model released | Update `model-catalog.md` only |
| New task type (e.g. "i18n reviewer") | Add row to `routing-matrix.md`; optional new template agent |
| Tool changes config format | Update `references/<tool>.md` + affected templates |
| User reports bad routing | Adjust tier in matrix, not per-platform files |

---

## 12. Success Criteria

The design is successful when:

1. A user can invoke the skill once and receive a coherent, tool-specific config set.
2. Changing a task's tier requires editing **one row** in `routing-matrix.md`, not four platform files.
3. Updating a deprecated model requires editing **one catalog**, not every template.
4. Existing user configs are detected and handled with explicit policy.
5. No marketing claims about guaranteed savings appear in skill or generated configs.

---

## 13. Open Questions

| # | Question | Default until decided |
|---|----------|----------------------|
| 1 | Ship as npm/curl installer or skill-folder-only? | Skill folder + GitHub |
| 2 | Include validation script (`scripts/validate-catalog.sh`)? | Phase 2+ optional |
| 3 | Windsurf tier mapping completeness | Document limitations honestly |
| 4 | Auto-detect profile from user's current model choices? | Manual profile for v1 |

---

## Appendix A: Example post-generation summary

```
Model routing configured for Cursor (project scope, balanced profile).

| Agent / task      | Tier               | Model                              |
|-------------------|--------------------|------------------------------------|
| explorer          | fast               | composer-2.5-fast                  |
| reviewer          | balanced-thinking  | claude-4.6-sonnet-medium-thinking  |
| tester            | balanced           | claude-4.6-sonnet-medium-thinking  |
| Built-in explore  | (product default)  | use for quick search               |
| Built-in shell    | (product default)  | use for terminal work              |

Files written:
  .cursor/rules/model-routing.mdc
  .cursor/agents/explorer.md
  .cursor/agents/reviewer.md
  .cursor/agents/tester.md

To adjust: edit references/routing-matrix.md tiers, or model-catalog.md IDs, then re-run the skill.
```

---

## Appendix B: README positioning copy

**Title:** Agent Model Optimizer

**Subtitle:** Task-appropriate model routing for AI coding agents.

**Body:**

> Configure subagents and delegated tasks to use the right model tier for each job — fast models for exploration and mechanical work, capable models for review and debugging, premium models for security and architecture. Works with Cursor, Claude Code, Codex, and Windsurf.
