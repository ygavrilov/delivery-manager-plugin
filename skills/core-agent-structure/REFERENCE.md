# Agent Structure Guide — Full Reference

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
