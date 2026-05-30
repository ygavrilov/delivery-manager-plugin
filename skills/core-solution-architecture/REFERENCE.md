# Solution Architecture — Full Reference

## Detailed Process Steps

### Step 1: Context review
- Agent: solution-architect
- Read PROJECT_CONTEXT.md, understand current stack and constraints
- Identify which layers are affected

### Step 2: Requirements clarification
- Agent: solution-architect + business-analyst
- Resolve ambiguities in requirements
- Identify implicit requirements (SEO, performance, security)
- Output: clarified requirement list

### Step 3: Options analysis
- Agent: solution-architect
- List 2–3 viable approaches
- For each: describe approach, list pros/cons, estimate complexity
- Apply KISS: prefer simplest option that meets all requirements
- Apply DRY: identify reuse opportunities but avoid premature abstraction

### Step 4: Technical deep-dive (per affected layer)
- Agent: solution-architect + relevant developer agents (as needed)
- Consult relevant expertise skills for project-specific patterns
- Define: DB schema changes, API contracts, frontend structure, infra changes
- Identify risks and dependencies

### Step 5: SEO & performance review (when applicable)
- Agent: solution-architect
- Evaluate impact on Core Web Vitals, crawlability, structured data
- Ensure meta tags, sitemaps, and schema markup are addressed
- Review analytics/tracking requirements (GTM, GA4, pixels)

### Step 6: Decision & documentation
- Agent: solution-architect
- Select recommended approach with justification
- Produce Architecture Decision Document
- List open questions for stakeholder review

---

## Output: Architecture Decision Document

```
# ADD: [Title]

## Problem Statement

One paragraph: what are we solving and why.

## Decision

Which approach was chosen and why.

## Options Considered

| Option | Description | Pros | Cons | Complexity   |
| ------ | ----------- | ---- | ---- | ------------ |
| A      |             |      |      | Low/Med/High |
| B      |             |      |      | Low/Med/High |

## Affected Layers

- **Database**: [migrations, schema changes, or "none"]
- **Backend**: [endpoints, services, or "none"]
- **Frontend**: [views, components, routes, or "none"]
- **Infrastructure**: [Docker, Nginx, caching, or "none"]
- **SEO/Analytics**: [meta, structured data, tracking, or "none"]

## API Contracts

[Request/response shapes for new/modified endpoints, or "N/A"]

## Risks & Mitigations

| Risk | Impact | Mitigation |
| ---- | ------ | ---------- |
|      |        |            |

## Open Questions

- [Items needing stakeholder input]
```

---

## RACI

- Solution outline
  - R: solution-architect
  - A: delivery-manager
  - C: developer agents (backend, frontend), business-analyst
  - I: stakeholders
