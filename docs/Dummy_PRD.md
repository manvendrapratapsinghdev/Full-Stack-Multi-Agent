# Dummy PRD — For Architecture Demo Only

> **This is a placeholder PRD used to demonstrate the agentic pipeline.**
> Replace with your actual PRD documents (e.g., `pitch.md`, `community_pr.md`).

---

## What goes here?

The PRD Agent ingests one or more product requirement documents and runs a 7-skill pipeline:

1. **PRD Parsing** — Break raw document into structured sections
2. **Feature Extraction** — Identify discrete features with IDs (F-001, F-002...)
3. **User Story Generation** — INVEST-compliant stories with acceptance criteria
4. **Gap Analysis** — Find missing requirements, safety holes, edge cases
5. **Business Goal Mapping** — Link features to KPIs
6. **Dependency Graphing** — DAG of feature dependencies via NetworkX
7. **Risk Assessment** — Technical, business, and operational risks

### Example Input Format

Your PRD should cover:

- Product vision and mission
- Problem statement
- Target users and personas
- Feature descriptions with user flows
- Business goals and KPIs
- Success metrics

### Output

The PRD Agent produces `docs/PRD_Analysis.md` containing:
- Structured feature cards (F-xxx) with MoSCoW priorities
- User stories (US-xxx) with Given/When/Then acceptance criteria
- Gap report (GAP-xxx)
- Risk register
- Dependency tiers
- Stakeholder clarifications needed

---

*Replace this file with your actual PRD to start the pipeline.*
