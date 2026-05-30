# Workflow Design — Full Step Reference

## Step 1: Clarify Process Intent

**Input:** Process description (raw)
**Output:** Clarified goal statement (1–2 sentences, unambiguous)
**Instructions:**
Ask the requester: "What problem does this workflow solve? When should someone run it? What does success look like?"
If the answer is already clear from the input, write the goal statement directly. Do not ask redundant questions.

---

## Step 2: Define Trigger Conditions

**Input:** Clarified goal statement
**Output:** Trigger definition — precise conditions under which this workflow should be initiated
**Instructions:**
Write trigger conditions as specific, observable events or states (e.g. "when a new project is added to PROJECT_CONTEXT.md" or "when a PRD is approved"). Avoid vague triggers like "when needed". If multiple triggers apply, list each one.

---

## Step 3: Define Expected Inputs

**Input:** Clarified goal statement + trigger conditions
**Output:** Input schema — a table of all inputs the workflow requires at start
**Instructions:**
For each input: name, required/optional, and a one-line description. Separate what the workflow needs to start from what it discovers during execution.

---

## Step 4: Identify Required Agents

**Input:** Clarified goal statement + input schema
**Output:** Agent list — which agents are needed and what each is responsible for
**Instructions:**
For each major action in the workflow, identify the best-fit agent from the available roster. Use agent descriptions to decide. If no existing agent covers a required action, flag it as a gap — do not invent an agent; escalate to the Delivery Manager.

---

## Step 5: Draft Steps (First Pass)

**Input:** Goal statement + inputs + agent list
**Output:** Rough ordered list of steps (not yet atomic)
**Instructions:**
Write every step the workflow needs to go from inputs to outcome. Focus on completeness, not atomicity. Order steps by dependency — a step should only appear after all its prerequisites.

---

## Step 6: Validate Step Atomicity

**Input:** Rough step list
**Output:** Atomic step list
**Instructions:**
Review each step. A step is atomic if:
- It produces exactly one output
- It can be completed by a single agent without needing to wait for another step mid-execution
If a step fails either check, split it. Repeat until all steps are atomic.

---

## Step 7: Define Step I/O

**Input:** Atomic step list
**Output:** Full step I/O table — for each step: input, output, exact agent instructions
**Instructions:**
For each atomic step, document:
- **Input**: what the agent receives at the start of this step
- **Output**: what the agent produces when the step is complete (the artifact or decision)
- **Instructions**: exact, actionable direction for the agent — specific enough that an agent with no prior context can execute it correctly

---

## Step 8: Define Workflow Outcome

**Input:** Full step I/O table + goal statement
**Output:** Outcome definition — what the completed workflow produces and how to verify it
**Instructions:**
Describe the final artifact(s) or state produced by running this workflow end-to-end. Define the acceptance criteria: how does the executor (or reviewer) confirm the workflow completed successfully?

---

## Step 9: Write the Workflow Skill File

**Input:** All outputs from Steps 1–8
**Output:** `workflow-<name>/SKILL.md` — a ready-to-use workflow skill
**Instructions:**
Create a new skill file using the standard structure:
1. YAML frontmatter: `name`, `description`
2. **When to Run** section (from Step 2)
3. **Expected Inputs** table (from Step 3)
4. **Required Agents** list (from Step 4)
5. **Steps** section — each step with Input, Output, Instructions (from Step 7)
6. **Outcome** section (from Step 8)

Name the file `workflow-<kebab-case-name>/SKILL.md` and place it in the `skills/` directory.

---

## Step 10: Review for Completeness

**Input:** Draft SKILL.md
**Output:** Approved SKILL.md (or list of revision items)
**Instructions:**
Check the following. Each item must pass before the workflow is considered done:
- [ ] Trigger conditions are specific and observable
- [ ] All required inputs are listed
- [ ] Each step is atomic (one output, one agent, no mid-step waits)
- [ ] Every step has explicit input, output, and instructions
- [ ] Required agents exist in the agent roster (no gaps unresolved)
- [ ] Outcome matches the original goal statement
- [ ] Skill file follows naming and formatting conventions

If any item fails, revise the relevant step(s) and re-check.
