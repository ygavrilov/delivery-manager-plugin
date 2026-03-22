---
name: workflow-solution-architecture
type: workflow
description: Process for creating solution architecture decisions. Defines the steps, inputs, outputs, and which sub-agents participate. Used by the delivery manager to orchestrate architecture work.
---

## Purpose

Produce an Architecture Decision Document (ADD) that defines how a feature or system change will be implemented across all affected layers.

## When to use

- New feature that touches multiple layers (backend, frontend, DB, infra)
- Technology choice or stack change evaluation
- Non-trivial integration (third-party services, APIs, analytics)
- Performance or SEO-related structural changes

## Inputs (required before starting)

1. **Requirements** — user stories or business requirements (from business-analyst)
2. **Project context** — `.claude/PROJECT_CONTEXT.md` (current stack, constraints, decisions)
3. **Scope boundary** — what is in/out for this decision

## Process (steps summary)

1. **Context review** — solution-architect reads PROJECT_CONTEXT.md, identifies affected layers
2. **Requirements clarification** — solution-architect + business-analyst resolve ambiguities
3. **Options analysis** — 2–3 viable approaches with pros/cons, apply KISS
4. **Technical deep-dive** — per affected layer with relevant developer agents
5. **SEO & performance review** — when applicable
6. **Decision & documentation** — select recommended approach, produce ADD

## Principles

- **KISS**: always recommend the simplest viable option
- **DRY**: identify true duplication; do not abstract for hypothetical reuse
- **One decision per ADD**: don't bundle unrelated decisions

For the full ADD output template and RACI, see `./SKILL-DETAILS.md`.
