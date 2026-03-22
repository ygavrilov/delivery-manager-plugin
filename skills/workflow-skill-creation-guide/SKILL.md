---
name: workflow-skill-creation-guide
type: workflow
description: Step-by-step guide for creating Claude skills
context: fork
---

# How to Create Claude Skills

## Skill Types

- **Task Skills:** Step-by-step instructions for specific actions.
- **Reference Skills:** Knowledge, conventions, style guides for your project.

## Best Practices

- Use meaningful names (becomes slash command)
- Write clear descriptions (include keywords users will say)
- Keep SKILL.md focused — move details to SKILL-DETAILS.md
- Reference supporting files when needed
- Limit tools with `allowed-tools` if needed
- Use `context: fork` for isolation
- Use `disable-model-invocation: true` for manual workflows

## Quick Start

1. Create a directory for your skill
2. Add SKILL.md with YAML frontmatter and instructions
3. Add SKILL-DETAILS.md for lengthy reference content
4. Add supporting files as needed (template.md, examples/, scripts/)

For the full file format, frontmatter field reference, string substitutions, storage locations, and advanced features, see `./SKILL-DETAILS.md`.
