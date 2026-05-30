# Skill Structure — Reference

## Categories

| Category | Prefix | Purpose |
|----------|--------|---------|
| `core` | `core-` | How we work: processes, practices, standards |
| `lang` | `lang-` | Language-specific patterns (php, typescript, etc.) |
| `framework` | `framework-` | Framework patterns (cakephp, vuejs, etc.) |
| `project` | `project-` | Per-project context; lives in target project's `.claude/` |

## Directory Structure

```
skills/
  {category}-{slug}/
    SKILL.md       ← frontmatter + 2-3 sentence summary
    REFERENCE.md   ← full content
```

Every skill has exactly these two files. No other files.

## SKILL.md Format

```yaml
---
name: {category}-{slug}
category: core | lang | framework | project
description: One clear sentence.
worker-hint: lead | senior | junior   # optional
---
```

Body: 2–3 sentences (what it covers, when to load it).
Final line: `See @REFERENCE.md for [what's in the reference file].`

### Frontmatter fields

| Field | Required | Notes |
|-------|----------|-------|
| `name` | yes | Must match directory name exactly |
| `category` | yes | `core` / `lang` / `framework` / `project` |
| `description` | yes | One sentence; used by orchestrator for skill selection |
| `worker-hint` | no | Guides orchestrator worker tier; not binding |

### Body rules

- 2–3 sentences maximum — no headings, no lists, prose only
- First sentence: what this skill covers
- Second sentence: when to load it
- Last line: `See @REFERENCE.md for ...` — use `@` so Claude resolves the file

## REFERENCE.md Format

- No frontmatter
- Plain markdown
- Core/workflow references: numbered steps with clear input/output
- Rules references: organized by domain, rationale where non-obvious
- Include concrete examples wherever a rule could be misread

## Naming

- Slug: lowercase, hyphen-separated, specific
- Examples: `core-requirements`, `framework-cakephp`, `lang-php`
- Frontmatter `name` must exactly match directory name
