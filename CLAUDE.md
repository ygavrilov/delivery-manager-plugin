# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A **Claude Code plugin** (`delivery-manager-plugin` v1.1.0) — a collection of agents and skills that gets installed into other projects via `.claude-plugin/`. It is not a runnable application; there is no build step, test suite, or linter.

The plugin's entry point is `settings.json`, which declares `"agent": "delivery-manager"` as the default agent for any project that installs it.

## Repository Structure

```
agents/          — Agent definition files (.md with YAML frontmatter)
skills/          — Skill definition files (SKILL.md in each subdirectory)
settings.json    — Plugin entry point: sets default agent to delivery-manager
.claude-plugin/plugin.json — Plugin metadata (name, version, description, author)
```

## Agent & Skill Architecture

**Agents** are role-based specialists with defined tool access. Each agent `.md` file has YAML frontmatter (`name`, `description`, `tools`, `context`, `color`, optional `skills`) followed by a system prompt.

**Skills** are reusable instructions loaded by agents. Two types:
- **Workflow skills** (`workflow-*`) — step-by-step guides for executing a specific process (e.g. `workflow-work-breakdown`, `workflow-solution-architecture`)
- **Knowledge skills** (`knowledge-*`) — reference material that dictates how things should be: patterns, templates, schemas (e.g. `knowledge-cakephp-architecture`, `knowledge-prd`)

### Agent Hierarchy

```
delivery-manager          ← workflow lifecycle owner: design, build, improve
├── project-context-manager   ← creates/maintains .claude/PROJECT_CONTEXT.md
├── project-manager           ← executes a specific project; tracks progress, coordinates agents
├── business-analyst          ← requirements gathering, stakeholder mapping
├── solution-architect        ← architecture decisions (no code); produces ADDs
├── cakephp-developer         ← backend implementation (CakePHP 5.0+)
├── database-architect        ← schema design, indexing strategy (MariaDB/MySQL)
├── vue-developer             ← admin SPA (Vue 3, Composition API, no UI libs)
└── devops                    ← VPS operations via SSH (nginx, SSL, deploy scripts, Docker)
```

**Delivery Manager vs Project Manager:**
- `delivery-manager` owns the workflow system — it designs new workflows, builds skills, and improves existing ones. It does not manage day-to-day project tasks.
- `project-manager` handles execution of a specific project — coordinating agents, tracking progress, resolving blockers within an established workflow.

### PROJECT_CONTEXT.md

The central artifact that all agents read on start. Located at `.claude/PROJECT_CONTEXT.md` in the **target project** (not this repo). Schema is defined in `skills/project-context/SKILL.md`. The `delivery-manager` creates it via `project-context-manager` if it doesn't exist.

## Adding or Modifying Content

### New agent
1. Create `agents/<name>.md` with YAML frontmatter: `name`, `description`, `tools`, `context` (fork), `color`
2. Write the system prompt body — include: what the agent does, what it doesn't do, how it works, and core principles
3. Register it in `agents/delivery-manager.md` under "Available Agents and Skills"

### New skill
1. Create `skills/<name>/SKILL.md` with YAML frontmatter: `name`, `description`
2. Write the skill content — process skills use numbered steps; expertise skills use reference patterns
3. Register it in `agents/delivery-manager.md` under the appropriate skill type

### Skill frontmatter convention
```yaml
---
name: skill-name
description: One-line description of what this skill provides.
---
```

### Agent frontmatter convention
```yaml
---
name: agent-name
description: One-line description used for agent selection.
tools: Read, Grep, Glob, Write, Bash   # only what the agent needs
context: fork                           # fork = isolated context
color: cyan                             # terminal display color
skills:                                 # optional: auto-loaded skills
    - skill-name
---
```

## Key Conventions

- **Agent responsibilities are strictly separated** — architects don't code, developers don't architect, business-analyst doesn't make technical decisions
- **All agents read `.claude/PROJECT_CONTEXT.md` on start** — this is the single source of truth for the target project
- **KISS and DRY** are stated principles in every agent; prefer simplest solution over clever abstraction
- **`vue-developer` uses plain HTML only** — no PrimeVue, Vuetify, or any UI component library (project decision)
- **`database-architect` produces DDL only** — actual CakePHP migration files are written by `cakephp-developer`
