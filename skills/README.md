# Shared Orchestration Skills

These skills are used by the orchestration-layer agents (not stack-specific).

---

## PRD Agent Skills

| Skill | Description | Input | Output |
|-------|-------------|-------|--------|
| `prd_parsing` | Parse raw PRD into 15+ structured sections | Raw markdown docs | Structured section map |
| `feature_extraction` | Extract discrete features with IDs | Parsed sections | Feature cards (F-xxx) |
| `user_story_generation` | Generate INVEST-compliant user stories | Feature cards | User stories (US-xxx) with Given/When/Then |
| `gap_analysis` | Identify missing requirements and safety holes | Feature cards + PRD | Gap report (GAP-xxx) |
| `business_goal_mapping` | Link features to KPIs and business objectives | Features + PRD goals | KPI mapping table |
| `dependency_graphing` | Build DAG of feature dependencies | Feature cards | Dependency tiers (NetworkX) |
| `risk_assessment` | Assess technical, business, operational risks | Full analysis | Risk register with severity |

## Master Orchestrator Skills

| Skill | Description |
|-------|-------------|
| `sprint_validation` | Validate sprint plan against capacity and dependency constraints |
| `contract_validation` | Run Spectral lint + openapi_diff on contract.yaml changes |
| `conflict_resolution` | Resolve cross-stack or cross-agent conflicts (auto -> human -> rollback) |
| `pipeline_enforcement` | Enforce mandatory 4-stage gate: Dev -> Code Review -> QA -> Commit |
| `state_management` | Update global_state.json with LangGraph checkpointing |
| `handshake_protocol` | Coordinate cross-stack contract sync between backend and frontend |

## Root Requirement Agent Skills

| Skill | Description |
|-------|-------------|
| `requirement_extraction` | Convert features to atomic TRD specs |
| `feasibility_assessment` | Evaluate technical feasibility per tech stack |
| `traceability_mapping` | Build RTM: PRD -> TRD -> Task -> Test |
| `conflict_detection` | Detect cross-stack API conflicts |
| `acceptance_criteria_gen` | Generate Gherkin-style Given/When/Then criteria |
| `change_impact_analysis` | Assess downstream impact of requirement changes |

## Task Breaking Agent Skills

| Skill | Description |
|-------|-------------|
| `requirement_decomposition` | Break TRD specs into atomic S/M/L/XL tasks |
| `sprint_sequencing` | Assign tasks to sprints with topological sort |
| `parallelism_detection` | Identify tasks that can run concurrently (DAG analysis) |
| `cross_stack_flagging` | Mark tasks requiring backend-frontend handshake |
| `scope_change_handling` | Re-plan affected tasks when scope changes |

## Progress Monitor Skills

| Skill | Description |
|-------|-------------|
| `velocity_tracking` | Rolling average velocity across last 3 sprints |
| `burndown_generation` | Generate sprint burndown/burnup metrics |
| `blocker_detection` | Flag tasks stuck > 2 days (warn) or > 3 days (escalate) |
| `standup_generation` | Produce daily standup summaries |

## Continuity Agent Skills

| Skill | Description |
|-------|-------------|
| `state_persistence` | PostgreSQL-backed checkpointing via LangGraph |
| `semantic_memory` | Qdrant vector DB for past decision retrieval |
| `session_handoff` | Preserve and restore context across sessions |
| `decision_logging` | Record Architecture Decision Records (ADRs) |
| `progressive_summarization` | 3-tier memory: working (< 1hr) / short-term (1-2wk) / long-term |
