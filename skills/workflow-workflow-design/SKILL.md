---
name: workflow-workflow-design
description: Meta-workflow for designing a new repeatable workflow. Produces a complete workflow definition: trigger conditions, expected inputs, required agents, atomic steps with I/O, and outcome.
---

# Workflow: Design a Workflow

## When to Run

Run this workflow when:
- A new repeatable process needs to be formalized so agents can execute it consistently
- An existing process is undocumented or inconsistently applied
- The Delivery Manager is asked to build a new workflow

Do **not** run this workflow for one-off tasks. A workflow is only worth designing if it will be reused.

## Expected Inputs

| Input | Required | Description |
|---|---|---|
| Process description | Yes | A plain-language description of the process to formalize |
| Goal statement | Yes | What the workflow should achieve (the "done" condition) |
| Existing documentation | No | Any prior notes, examples, or ad-hoc steps already in use |
| Constraints | No | Known limitations (agents available, tools, time) |

## Required Agents

- **Delivery Manager** — orchestrates this workflow
- **Business Analyst** — clarifies process intent and steps (invoke for Steps 1–3)
- **Solution Architect** — optional; needed only if the workflow involves technical architecture decisions

## Steps (summary)

1. Clarify process intent → clarified goal statement
2. Define trigger conditions → specific, observable triggers
3. Define expected inputs → input schema table
4. Identify required agents → agent list with responsibilities
5. Draft steps (first pass) → rough ordered list
6. Validate step atomicity → each step: one output, one agent, no mid-step waits
7. Define step I/O → full step table with input, output, instructions
8. Define workflow outcome → final artifacts and acceptance criteria
9. Write the workflow skill file → `workflow-<name>/SKILL.md`
10. Review for completeness → checklist pass before done

## Outcome

A complete `workflow-<name>/SKILL.md` file that can be immediately loaded and executed without additional clarification.

For full step-by-step instructions and the Step 10 review checklist, see `./SKILL-DETAILS.md`.
