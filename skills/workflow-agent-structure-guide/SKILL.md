---
name: workflow-agent-structure-guide
type: workflow
description: Step-by-step guide for structuring Claude subagents
context: fork
---

# How to Structure Claude Subagents

## Subagent Purpose

- Specialized AI assistants for focused tasks
- Preserve context, enforce constraints, reuse configs, specialize behavior, control costs

## Best Practices

- Design focused subagents (one task per agent)
- Write clear, actionable descriptions (Claude uses this to delegate)
- Limit tool access for security and focus
- Check into version control for project subagents
- Use detailed system prompts for domain-specific behavior

## Quick Start

1. Create a Markdown file with YAML frontmatter for your subagent
2. Add a clear description and system prompt
3. Specify tool access and advanced fields as needed
4. Store in the appropriate directory for scope and priority

For the full file format, frontmatter field reference, storage location table, and advanced features, see `./SKILL-DETAILS.md`.
