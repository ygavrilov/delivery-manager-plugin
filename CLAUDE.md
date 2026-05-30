# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A **Claude Code plugin** (`delivery-manager-plugin` v2.0.0) — a collection of agents and skills that gets installed into other projects via `.claude-plugin/`. It is not a runnable application; there is no build step, test suite, or linter.

The plugin's entry point is `settings.json`, which declares `"agent": "delivery-manager"` as the default agent for any project that installs it.

## Repository Structure

```
agents/          — Agent definitions (.md with YAML frontmatter)
skills/          — Skill directories (each contains SKILL.md + REFERENCE.md)
settings.json    — Plugin entry point: sets default agent to delivery-manager
.claude-plugin/plugin.json — Plugin metadata (name, version, description, author)
```

## Architecture: Orchestrator / Worker

**Delivery Manager** is the orchestrator. It owns context, selects skills, and dispatches tasks to workers.

**Workers** are execution agents — generic, forked, model-tiered:

| Agent | Model | Use for |
|-------|-------|---------|
| `hard-worker` | opus | Complex analysis, architecture, critical implementation |
| `regular-worker` | sonnet | Standard tasks, documentation, routine implementation |
| `laid-back-worker` | haiku | Simple edits, lookups, formatting, summarization |

All workers use `context: fork` — they see only what the orchestrator passes them.

## Skills

Two categories. Every skill is a directory with exactly two files:

```
skills/{category}-{slug}/
  SKILL.md       ← frontmatter + brief summary (2–3 sentences)
  REFERENCE.md   ← full content (steps, rules, examples, templates)
```

| Category | Prefix | Purpose |
|----------|--------|---------|
| `workflow` | `workflow-` | Step-by-step process execution guides |
| `rules` | `rules-` | Guidelines, patterns, and constraints for a type of work |

### Skill frontmatter convention

```yaml
---
name: {category}-{slug}       # must match directory name
category: workflow | rules
description: One clear sentence.
worker-hint: hard | regular | laid-back   # optional — guidance for orchestrator
---
```

## Adding or Modifying Content

### New worker agent
1. Create `agents/{name}.md` with frontmatter: `name`, `description`, `model`, `tools`, `context: fork`, `color`
2. Write minimal system prompt: on-start steps, execution rules, what it does not do
3. Register in `agents/delivery-manager.md` Workers table

### New skill
1. Follow `rules-skill-master` — authoritative convention for skill structure
2. Create `skills/{category}-{slug}/SKILL.md` with frontmatter + 2–3 sentence summary
3. Create `skills/{category}-{slug}/REFERENCE.md` with full content
4. Register in `agents/delivery-manager.md` under appropriate skill category

## Key Conventions

- **Orchestrator owns context** — delivery-manager does not fork; workers always fork
- **Skills are the knowledge layer** — role-specific patterns live in rules-* skills, not in agents
- **SKILL.md is light** — frontmatter + brief summary only; all detail in REFERENCE.md
- **rules-skill-master is authoritative** — follow it when creating or updating any skill
- **KISS and DRY** — simplest solution, no speculative abstractions
