# Delivery Manager Plugin

Claude Code plugin v3 — orchestrated project delivery via engineer workers and domain skills.

## What It Does

Installs a `delivery-manager` orchestrator into any Claude Code project. The orchestrator dispatches tasks to model-tiered engineer agents, guided by a skill library covering how we work, languages, and frameworks.

## Structure

```
agents/         delivery-manager + lead/senior/junior engineer
skills/
  core-*        how we work: processes, practices, standards
  lang-*        language patterns (php, typescript, ...)
  framework-*   framework patterns (cakephp, vuejs, ...)
  project-*     per-project context (lives in target project's .claude/)
```

## Agents

| Agent | Model | Role |
|-------|-------|------|
| `delivery-manager` | — | Orchestrator. Reads context, selects skills, dispatches. |
| `lead-engineer` | opus | Complex analysis, architecture, critical implementation. |
| `senior-engineer` | sonnet | Standard implementation, docs, moderate analysis. |
| `junior-engineer` | haiku | Minor edits, lookups, formatting. |
| `semble-search` | — | Semantic code search via semble CLI. |

## Key Skills

| Skill | Purpose |
|-------|---------|
| `core-ingest-repo` | Onboard an existing project → produces `PROJECT_CONTEXT.md` |
| `core-framework` | Core principles guiding all decisions |
| `core-skill-structure` | Authoritative convention for authoring skills |
| `framework-cakephp` | CakePHP patterns and conventions |
| `framework-vuejs` | Vue.js patterns and conventions |

## Code Search

Semble MCP + sub-agent integrated for token-efficient repo exploration. Add once at user level:

```bash
claude mcp add semble -s user -- uvx --from "semble[mcp]" semble
```

## Install

Add to a project's Claude Code settings as a plugin pointing to this repo.
