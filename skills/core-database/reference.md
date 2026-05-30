# Database Design Rules

## Schema Design Approach

1. Read PROJECT_CONTEXT.md — understand current data model, entities, and relationships
2. Clarify requirements if ambiguous: relationships, expected data volume, query patterns
3. Normalize to 3NF unless denormalization is explicitly justified
4. Choose column types per rules below
5. Define all constraints: primary key, unique, foreign keys, not-null, defaults
6. Define indexes: query-driven only
7. Follow CakePHP ORM naming conventions
8. Present schema as DDL with reasoning for every non-obvious decision

## Column Type Rules

| Purpose | Type |
|---------|------|
| Boolean | `tinyint(1)` |
| Short strings | `varchar` with sensible max length |
| Long content | `text` |
| Sort orders, FKs | `int` |
| Timestamps | `datetime` |

Always specify: type, length, nullability, and default value. No bare `varchar` without length.

## CakePHP ORM Naming

- Table names: plural, snake_case (`user_roles`, `blog_posts`)
- Primary key: `id`, auto-increment integer
- Foreign keys: `{singular_table}_id` (e.g. `user_id`, `post_id`)
- Timestamps: `created` and `modified` datetime columns on every table
- Junction tables: alphabetical order (`posts_tags` not `tags_posts`)

## Index Discipline

- Index only what queries need — every unused index is a write-time penalty
- Index all foreign key columns
- Composite indexes: most selective column first
- Use unique indexes to enforce business uniqueness constraints at DB level
- No speculative indexes "for future queries"

## Migration Safety Rules

- Design for safe forward-only migrations
- Avoid changes that require data backfill unless unavoidable
- When backfill is required: add nullable column → backfill → add not-null constraint in separate migration
- Never rename columns in place on live tables — add new, migrate data, drop old
- Output: DDL statements as design artifact; CakePHP migration file is written separately

## Principles

- **KISS**: simplest schema that satisfies requirements
- **CakePHP conventions**: follow ORM naming so no custom mapping is needed in application code
- **Explicit types**: always specify column type, length, nullability, and default
- **Index discipline**: index for actual query patterns, not speculation
- **Migrations are additive**: design for forward-only; backfill only when unavoidable
