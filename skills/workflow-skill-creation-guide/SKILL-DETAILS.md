# Skill Creation Guide — Full Reference

## Skill File Format

A skill directory must contain:

- SKILL.md (main instructions, required)
- SKILL-DETAILS.md (extended reference content, referenced from SKILL.md)
- template.md (optional)
- examples/ (optional)
- scripts/ (optional)

### SKILL.md Structure

1. YAML frontmatter between `---` markers
2. Markdown instructions

#### Example

```yaml
---
name: workflow-skill-creation-guide
type: workflow
description: Migrate a component from one framework to another
---
Migrate the $0 component from $1 to $2.
Preserve all existing behavior and tests.
```

## Frontmatter Fields

| Field                    | Description                              |
| ------------------------ | ---------------------------------------- |
| name                     | Display name and slash command           |
| description              | What the skill does and when to use it   |
| argument-hint            | Autocomplete hint for expected arguments |
| disable-model-invocation | Prevents auto-loading                    |
| user-invocable           | Hide from / menu                         |
| allowed-tools            | Tools Claude can use                     |
| model                    | Model to use                             |
| context                  | Set to fork for subagent                 |
| agent                    | Subagent type                            |
| hooks                    | Lifecycle hooks                          |

## String Substitutions

- `$ARGUMENTS` for all arguments
- `$N` for argument by index
- `${CLAUDE_SESSION_ID}` for session ID

## Storage Locations

| Location   | Path                                   | Scope                   |
| ---------- | -------------------------------------- | ----------------------- |
| Enterprise | Managed settings                       | Organization-wide       |
| Personal   | ~/.claude/skills/<skill-name>/SKILL.md | All projects            |
| Project    | .claude/skills/<skill-name>/SKILL.md   | This project only       |
| Plugin     | <plugin>/skills/<skill-name>/SKILL.md  | Where plugin is enabled |

## Advanced Features

- Dynamic context injection with `` !`command` ``
- Tool restrictions with `allowed-tools`
- Invocation control with `disable-model-invocation` and `user-invocable`
