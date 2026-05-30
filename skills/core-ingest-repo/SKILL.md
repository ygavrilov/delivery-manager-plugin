---
name: core-ingest-repo
category: core
description: Analyze an existing project repo and produce a populated PROJECT_CONTEXT.md.
worker-hint: lead
---

Covers four analysis dimensions — technology, architecture, standards, state — and produces a populated `.claude/PROJECT_CONTEXT.md` in the target project. Load when onboarding an existing project into the plugin system.

See `REFERENCE.md` for analysis steps, what to look for in each dimension, and output format.
