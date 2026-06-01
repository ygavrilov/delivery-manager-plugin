# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What This Repo Is

A **Claude Code plugin** (`delivery-manager-plugin` v3.0.0) — agents and skills installed into other projects via `.claude-plugin/`. Not a runnable application; no build step, test suite, or linter.

Plugin entry point: `settings.json` → declares `"agent": "delivery-manager-plugin:delivery-manager"` as default agent.

## Repository Structure

```
agents/                    — Agent definitions (.md with YAML frontmatter)
skills/                    — Skill directories (each: SKILL.md + REFERENCE.md)
settings.json              — Plugin entry point
.claude-plugin/plugin.json — Plugin metadata (name, version, author)
.claude/agents/            — Local sub-agents (semble-search)
```

## Architecture: Orchestrator / Engineers

**Delivery Manager** is the orchestrator — owns context, selects skills, dispatches to engineers.

**Engineers** are execution agents — forked, model-tiered:

| Agent | Model | Use for |
|-------|-------|---------|
| `lead-engineer` | opus | Architecture, complex analysis, critical implementation |
| `senior-engineer` | sonnet | Standard implementation, docs, moderate analysis |
| `junior-engineer` | haiku | Minor edits, lookups, formatting, summarization |

All engineers use `context: fork` — see only what orchestrator passes them.

## Skills

Four categories. Every skill is a directory with exactly two files:

```
skills/{category}-{slug}/
  SKILL.md       ← required: overview + navigation (under 500 lines)
  reference.md   ← detailed content — loaded on demand
  examples.md    ← examples — loaded on demand (optional)
  scripts/       ← utility scripts — executed, not loaded (optional)
```

| Category | Prefix | Purpose | Location |
|----------|--------|---------|----------|
| `core` | `core-` | How we work: processes, practices, standards | plugin |
| `lang` | `lang-` | Language-specific patterns (php, typescript) | plugin |
| `framework` | `framework-` | Framework patterns (cakephp, vuejs) | plugin |
| `project` | `project-` | Per-project context | target project's `.claude/` |

Every skill is also one of two **types** (`type:` frontmatter): `workflow` (the Delivery Manager runs it) or `rule` (a worker applies it). Skills are preloaded into agents via the `skills:` frontmatter; because each SKILL.md is skinny, agents select the right skill by its description and pull `reference.md` on demand.

### Skill frontmatter

```yaml
---
name: {category}-{slug}       # must match directory name exactly
category: core | lang | framework | project
type: workflow | rule         # workflow = DM runs it; rule = worker applies it
description: One clear sentence.
worker-hint: lead | senior | junior   # optional
---
```

## Code Search

Semble MCP is configured at user level (`search`, `find_related` tools). Use it for semantic code exploration — far cheaper than grep+read.

```bash
# CLI (workers use this)
semble search "authentication flow" ./path/to/repo
semble search "UserController" . --top-k 5
semble find-related src/Controller/Users.php 42 .

# MCP (delivery-manager uses this directly)
search(query="entry points routing", repo="/path/to/repo")
find_related(file_path="src/auth.php", line=12, repo="/path/to/repo")
```

**Rule:** semble before grep. Grep only for exhaustive exact-string matches.

`semble-search` sub-agent available at `.claude/agents/semble-search.md` for dedicated search tasks.

## Adding or Modifying Content

### New engineer agent
1. Create `agents/{name}.md` — frontmatter: `name`, `description`, `model`, `tools`, `context: fork`, `color`
2. Minimal system prompt: on-start steps, code search guidance, execution rules, what it does not do
3. Register in `agents/delivery-manager.md` Workers table

### New skill
1. Follow `core-skill-structure` — authoritative convention
2. Create `skills/{category}-{slug}/SKILL.md` — frontmatter + 2–3 sentence summary
3. Create `skills/{category}-{slug}/REFERENCE.md` — full content
4. Register in `agents/delivery-manager.md` under the matching `type` group (Workflows / Rules)
5. Add the skill name to the `skills:` frontmatter of the agents that need it (workflows → `delivery-manager`; rules → the three engineers)

## Key Conventions

- **Orchestrator owns context** — delivery-manager does not fork; engineers always fork
- **Specialist roles = skills** — BA, architect, QA knowledge lives in skills, not agents
- **SKILL.md is light** — frontmatter + summary only; all detail in REFERENCE.md
- **`core-skill-structure` is authoritative** — follow it for any skill work
- **KISS and DRY** — simplest solution, no speculative abstractions
