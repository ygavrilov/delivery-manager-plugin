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

## Subagent File Format

A subagent directory must contain:

- Subagent file (Markdown with YAML frontmatter, required)

### Subagent File Structure

1. YAML frontmatter between `---` markers
2. System prompt in Markdown

#### Example

```markdown
---
name: workflow-agent-structure-guide
type: workflow
description: Reviews code for quality and best practices
tools: Read, Glob, Grep
model: sonnet
---

You are a code reviewer. When invoked, analyze the code and provide specific, actionable feedback on quality, security, and best practices.
```

## Frontmatter Fields

| Field           | Required | Description                                          |
| --------------- | -------- | ---------------------------------------------------- |
| name            | Yes      | Unique identifier (lowercase, hyphens)               |
| description     | Yes      | When Claude should delegate to this subagent         |
| tools           | No       | Allowed tools (inherits all if omitted)              |
| disallowedTools | No       | Tools to deny                                        |
| model           | No       | Model to use (`sonnet`, `opus`, `haiku`, `inherit`)  |
| permissionMode  | No       | Permission mode (`default`, `acceptEdits`, etc.)     |
| maxTurns        | No       | Max agentic turns                                    |
| skills          | No       | Skills to preload into context                       |
| mcpServers      | No       | MCP servers available                                |
| hooks           | No       | Lifecycle hooks                                      |
| memory          | No       | Persistent memory scope (`user`, `project`, `local`) |
| background      | No       | Run as background task                               |
| isolation       | No       | Run in isolated git worktree                         |

## Subagent Storage Locations

| Location | Path                             | Scope                   | Priority |
| -------- | -------------------------------- | ----------------------- | -------- |
| CLI flag | --agents                         | Current session         | 1        |
| Project  | .claude/agents/<agent-name>.md   | This project only       | 2        |
| Personal | ~/.claude/agents/<agent-name>.md | All projects            | 3        |
| Plugin   | <plugin>/agents/<agent-name>.md  | Where plugin is enabled | 4        |

## Advanced Features

- Restrict tool access with `tools` and `disallowedTools`
- Control which subagents can be spawned with `Agent(agent_type)`
- Set permission modes for workflow control
- Preload skills for domain knowledge
- Enable persistent memory for cross-session learning
- Run as background or in isolated worktree

## Quick Start

1. Create a Markdown file with YAML frontmatter for your subagent
2. Add a clear description and system prompt
3. Specify tool access and advanced fields as needed
4. Store in the appropriate directory for scope and priority

(See <attachments> above for file contents. You may not need to search or read the file again.)
