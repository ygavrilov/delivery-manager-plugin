# Ingest Repo — Reference

## Goal

Produce a populated `.claude/PROJECT_CONTEXT.md` for an existing project. No prior project context exists — derive everything from the codebase.

## Output

`.claude/PROJECT_CONTEXT.md` following the schema in `core-project-context`.

---

## Analysis Dimensions

### 1. Technology

What is the project built with?

- **Language(s)**: PHP version, Node/TS version, etc.
- **Framework(s)**: CakePHP, Laravel, Vue, React, etc.
- **Key dependencies**: check `composer.json`, `package.json`, `requirements.txt`
- **Infrastructure**: Docker, nginx, VPS, cloud — check `docker-compose.yml`, CI files, deployment scripts
- **Database**: engine, ORM, migration tool

### 2. Architecture

How is the project structured?

- **Pattern**: MVC, monolith, API + SPA, microservices, etc.
- **Entry points**: where requests enter (controllers, routes, index files)
- **Layers**: where business logic lives (services, domain, models)
- **Frontend**: SPA, SSR, blade/twig templates, separate repo?
- **APIs**: REST, GraphQL, internal only?

### 3. Standards

How is code written?

- **Directory structure**: where do models, controllers, services, views live?
- **Naming conventions**: file names, class names, method names
- **Config pattern**: env files, config files, secrets management
- **Testing approach**: test framework, coverage, conventions

### 4. State

What is the current condition of the project?

- **Phase**: active development, maintenance, legacy, abandoned?
- **Health indicators**: recent commits, open issues, test pass rate
- **Known issues**: anything flagged in README, comments, or obvious from code
- **Deployment state**: running in production? where?

---

## Steps

1. **Locate entry points** — `README.md`, `composer.json`/`package.json`, `Makefile`, CI config
2. **Map technology** — language + framework + key deps + infra
3. **Trace architecture** — follow a request from entry point to response; identify layers
4. **Read standards** — sample 3–5 files per layer; note naming and structure patterns
5. **Assess state** — `git log --oneline -20`, check for TODOs, check test presence
6. **Confirm with user** — summarize findings in 5–7 bullets; ask to confirm before writing
7. **Write PROJECT_CONTEXT.md** — use confirmed findings; follow `core-project-context` schema

---

## Rules

- Confirm before writing — step 6 is mandatory; do not write context without user confirmation
- One-liners only in the output file — no prose
- Stack: only confirmed tech; never speculative
- If a dimension is unclear, say so in Open Questions, do not guess
