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
    SKILL.md       ← required: overview + navigation (under 500 lines)
    reference.md   ← detailed content — loaded on demand
    examples.md    ← usage examples — loaded on demand (optional)
    scripts/       ← utility scripts — executed, not loaded (optional)
```

SKILL.md is required. All other files are optional and loaded only when referenced.

## SKILL.md Format

```yaml
---
name: {category}-{slug}
category: core | lang | framework | project
description: One clear sentence.
worker-hint: lead | senior | junior   # optional
---
```

Body: 2–3 sentences (what it covers, when to load it). No headings, no lists, prose only.

Then an `## Additional Resources` section linking to supporting files:

```markdown
## Additional Resources

- For [description of content], see [reference.md](reference.md)
- For usage examples, see [examples.md](examples.md)
```

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
- Then `## Additional Resources` section with markdown links to supporting files

### Additional Resources rules

- Use standard markdown links: `[reference.md](reference.md)`
- Do NOT use `@filename` — links are for on-demand loading, not auto-include
- Describe what each file contains so Claude knows when to load it
- Keep SKILL.md under 500 lines; move detail to reference.md

## reference.md Format

- No frontmatter
- Plain markdown
- Core skill references: numbered steps with clear input/output
- Rules references: organized by domain, rationale where non-obvious
- Include concrete examples wherever a rule could be misread

## Naming

- Slug: lowercase, hyphen-separated, specific
- Supporting files: lowercase (reference.md, examples.md, not REFERENCE.md)
- Frontmatter `name` must exactly match directory name
- Examples: `core-requirements`, `framework-cakephp`, `lang-php`
