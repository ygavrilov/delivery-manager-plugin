# Delivery Manager Plugin

Claude Code plugin v3 — orchestrated project delivery via engineer workers and domain skills.

## Phase

Development (v3 migration in progress)

## Stack

- Markdown (agent/skill definitions)
- YAML frontmatter (agent/skill metadata)
- Claude Code plugin system (`.claude-plugin/plugin.json`)

## Agents & Skills

Agents: `delivery-manager` (orchestrator), `lead-engineer` (opus), `senior-engineer` (sonnet), `junior-engineer` (haiku)
Core skills (21): ingest-repo, framework, sdlc, project-context, rfq, requirements, solution-architecture, work-breakdown, specification, progress-tracker, testing, architecture, database, prd, rfd, vps, agent-structure, skill-structure, workflow-design
Framework skills: framework-cakephp, framework-vuejs

## Key Paths

- `agents/delivery-manager.md` — orchestrator (skills registry, dispatch rules)
- `agents/lead-engineer.md` / `senior-engineer.md` / `junior-engineer.md` — worker tiers
- `skills/core-*/` — how we work (processes, practices, standards)
- `skills/framework-*/` — framework patterns
- `skills/lang-*/` — language patterns (none yet; created as needed)
- `skills/project-*/` — per-project; stored in target project's `.claude/`
- `.claude-plugin/plugin.json` — plugin metadata (v3.0.0)

## Constraints & Decisions

- Orchestrator never forks; workers always fork (`context: fork`)
- Specialist roles (BA, architect, etc.) encoded as skills, not agents
- SKILL.md = frontmatter + 2-3 sentence summary; all detail → REFERENCE.md
- `core-skill-structure` is authoritative for skill authoring
- 4 skill categories: core / lang / framework / project
- `project-*` skills live in target project's `.claude/`, not in plugin
- `core-ingest-repo` is entry point for existing project onboarding

## Open Questions

- `lang-*` skills: create lang-php and lang-typescript when first needed
