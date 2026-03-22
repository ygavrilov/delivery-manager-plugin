---
name: workflow-requirements-details
description: Full step-by-step detail for the requirements workflow — interview questions, stakeholder mapping table, process analysis, and PRD/RFD completion instructions.
---

# Workflow: Requirements Gathering — Full Detail

## Step 1: Classify the Request

Ask yourself (or the requester):
- Does this add, change, or remove something a user directly interacts with? → **PRD**
- Is this a technical change, integration, migration, or system improvement? → **RFD**

If unclear, lean toward PRD. You can always reclassify after the interview.

---

## Step 2: Structured Requirements Interview

Ask the following questions. You do not need to ask all of them mechanically — adapt to what the requester has already told you. Fill in every field before producing the output document.

### Core questions (always ask)

1. **What problem does this solve?** Who experiences the problem, and how often?
2. **Who are the primary users?** What are they trying to accomplish?
3. **What does success look like?** How will we know this is working?
4. **What is explicitly out of scope?** What should this NOT do?
5. **Are there hard constraints?** (deadline, tech stack, budget, regulation, existing system limits)

### Clarifying questions (ask when the answer is unclear)

6. **What does the current experience / process look like?** Walk me through it step by step.
7. **What breaks or is painful in the current process?** Where does time get lost?
8. **Are there other teams or systems affected?** Who else needs to be consulted?
9. **Is there existing documentation, mockups, or prior attempts?** If so, share them.
10. **What is the priority relative to other work?** Is there a deadline?

### Open questions to capture

Note anything the requester cannot answer. These go into the **Open Questions** section of the output document and must be resolved before development begins.

---

## Step 3: Stakeholder Mapping (Inline)

While conducting the interview, build the stakeholder table. You do not need a separate skill for this — do it here.

| Stakeholder | Role | Interest in this work | Influence | Action |
|-------------|------|-----------------------|-----------|--------|
| [Name/Team] | [Role] | [What they care about] | High/Med/Low | Inform / Consult / Approve |

Rules:
- List every person or team mentioned during the interview.
- Mark anyone who must approve the final document as **Approve**.
- Mark teams affected by the change as **Inform**.
- If a stakeholder's needs conflict with another's, flag it in Open Questions.

---

## Step 4: Process Analysis (Inline)

Document the current state before defining the future state. Keep this concise.

**Current process:**
1. [Step 1]
2. [Step 2]
3. ...

**Pain points:**
- [What is slow, error-prone, or manual]

**Future process (proposed):**
1. [Step 1]
2. [Step 2]
3. ...

**Delta:** What changes between current and future? This delta drives the Requirements section of the output document.

---

## Step 5: Produce the Output Document

### If PRD (user-facing feature):

Use the `knowledge-prd` template. Fill in every section:

- **Problem / Opportunity**: one paragraph using the answer to interview question 1.
- **Users**: from question 2.
- **Scope In/Out**: from questions 2 and 4.
- **Requirements**: derive from the delta in Step 4, expressed as user-observable outcomes ("The user can…", "The system will…").
- **Success Criteria**: from question 3 — measurable, binary where possible.
- **Open Questions**: every unanswered question from Step 2 and Step 3.

### If RFD (technical improvement):

Use the `knowledge-rfd` template. Fill in every section:

- **Problem Statement**: one paragraph using questions 1 and 6.
- **Proposed Change**: what will be done — no implementation detail yet, just intent.
- **Scope In/Out**: from questions 2 and 4.
- **Constraints**: from question 5.
- **Acceptance Criteria**: from question 3 — verifiable conditions.
- **Open Questions**: every unanswered question from Step 2 and Step 3.

---

## Definition of Done

- [ ] Request classified as PRD or RFD
- [ ] All core interview questions answered (or captured as Open Questions)
- [ ] Stakeholder table complete
- [ ] Current process documented
- [ ] Output document filled in with no empty sections (use "None" if a section genuinely has nothing)
- [ ] Open Questions listed and owners identified where possible
