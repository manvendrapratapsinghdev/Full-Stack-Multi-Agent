# Multi-Agent Orchestration Architecture

## What is this?

This folder contains the **complete agentic development framework** used to build a full-stack application (Flutter + FastAPI) using **16 specialized AI agents** coordinated by a master orchestrator. No application source code is included — only the agent definitions, skills, communication protocols, and documentation artifacts.

**Share this folder with your team to understand the agentic workflow without needing the codebase.**

---

## Folder Structure

```
architecture/
|
|-- README.md                       # You are here
|-- global_state.json               # Project-wide state tracker
|-- communication_schema.json       # Inter-agent message protocol (26 message types)
|-- contract.yaml                   # OpenAPI 3.0.3 API contract (single source of truth)
|-- flow_diagram.mmd                # Mermaid flowchart of full orchestration flow
|-- agent_architecture.html         # Interactive visual diagram (open in browser)
|
|-- agents/                         # SHARED ORCHESTRATION AGENTS (6)
|   |-- master_orchestrator/        # Principal coordinator
|   |-- prd_agent/                  # PRD analysis & feature extraction
|   |-- requirement_agent/          # TRD formalization & traceability
|   |-- task_breaking_agent/        # Sprint planning & task decomposition
|   |-- progress_monitor/           # Velocity tracking & blocker detection
|   |-- continuity_agent/           # Context preservation across sessions
|
|-- skills/                         # Shared agent skill definitions
|   |-- README.md
|
|-- docs/                           # Project-level documentation
|   |-- Dummy_PRD.md                # Placeholder PRD (replace with yours)
|   |-- PRD_Analysis.md             # Output: 38 features, 22 gaps, 45 stories
|   |-- Master_TRD.md               # 32 TRD specs across 10 modules
|   |-- Requirement_Traceability_Matrix.md  # PRD -> Feature -> TRD -> Task mapping
|   |-- Sprint_Task_List.md         # 108 tasks across 10 sprints
|
|-- app/                            # FRONTEND STACK (Flutter)
|   |-- agents/                     # Flutter agents (5)
|   |   |-- flutter_requirement_agent/   # Contract -> Dart model generation
|   |   |-- flutter_dev_lead/            # Feature implementation
|   |   |-- flutter_code_review/         # Architecture/security/UI review gate
|   |   |-- flutter_qa/                  # Widget/integration/golden testing
|   |   |-- flutter_commit_agent/        # Git commit with gate enforcement
|   |-- skills/                     # Flutter skill definitions
|   |   |-- README.md
|   |-- docs/                       # Flutter-specific docs
|       |-- App_TRD.md
|       |-- App_Task_List.md
|
|-- backend/                        # BACKEND STACK (FastAPI)
    |-- agents/                     # Python agents (5)
    |   |-- python_requirement_agent/    # OpenAPI contract authoring
    |   |-- python_dev_lead/             # FastAPI implementation
    |   |-- python_code_review/          # Security/performance review gate
    |   |-- python_qa/                   # Unit/contract/load testing
    |   |-- python_commit_agent/         # Git commit with gate enforcement
    |-- skills/                     # Backend skill definitions
    |   |-- README.md
    |-- docs/                       # Backend-specific docs
        |-- Backend_TRD.md
        |-- Backend_Task_List.md
```

---

## How the Pipeline Works

### Phase 1: PRD Analysis
`PRD Documents` -> `PRD_Agent` -> `PRD_Analysis.md` (38 features, 22 gaps, 45 user stories)

### Phase 2: Requirement Formalization
`PRD_Analysis` -> `Root_Requirement_Agent` -> `Master_TRD.md` + `Traceability_Matrix.md`

### Phase 3: Task Decomposition
`Master_TRD` -> `Task_Breaking_Agent` -> `Sprint_Task_List.md` (108 tasks, 10 sprints)

### Phase 4: Orchestration
`Sprint Plan` -> `Master_Orchestrator` -> validates contract -> dispatches tasks to stacks

### Phase 5: Parallel Stack Execution
Both stacks run simultaneously with a **mandatory 4-stage pipeline**:

```
Dev Lead -> Code Review (BLOCKING GATE) -> QA (BLOCKING GATE) -> Commit Agent
```

**No code can be committed without passing BOTH code review AND QA.**

### Phase 6: Sprint Completion
Both stacks merge -> Master updates state -> next sprint begins

---

## Key Concepts

### Mandatory Pipeline Gates
Every task goes through 4 stages. Skipping is impossible:
1. **Dev** completes implementation
2. **Code Review** must PASS (architecture, security, performance, quality)
3. **QA** must PASS (unit, integration, contract, load tests)
4. **Commit Agent** creates the commit only after both gates pass

### Cross-Stack Handshake Protocol
When the backend API contract changes:
1. `Python_Requirement_Agent` updates `contract.yaml`
2. `Master_Orchestrator` validates with Spectral lint
3. `HANDSHAKE_INIT` sent to `Flutter_Requirement_Agent`
4. Flutter re-generates Dart models from updated contract
5. `HANDSHAKE_ACK` confirms sync is complete

### Communication Protocol
- **Transport:** Redis PubSub (async, event-driven)
- **26 message types** defined in `communication_schema.json`
- **UUID-based** message tracking with correlation IDs
- **Priority levels:** CRITICAL / HIGH / MEDIUM / LOW

### State Management
- `global_state.json` tracks overall project status
- Per-agent thought logs record decisions and rationale
- LangGraph checkpointing for session persistence
- 3-tier memory: working (< 1hr) / short-term (1-2wk) / long-term (permanent)

### Full Traceability
Every line of code traces back through the chain:
```
PRD Document -> Feature Card (F-xxx) -> TRD Spec (TRD-MODULE-NNN) -> Sprint Task (S{n}-{seq}) -> Git Commit -> Test Case
```

---

## Agent Summary (16 Total)

| Layer | Agent | Role |
|-------|-------|------|
| **Orchestration** | Master_Orchestrator | Principal coordinator, pipeline enforcer |
| | PRD_Agent | PRD analysis, feature extraction, gap detection |
| | Root_Requirement_Agent | TRD formalization, traceability |
| | Task_Breaking_Agent | Sprint planning, task decomposition |
| | Progress_Monitor | Velocity tracking, blocker detection |
| | Continuity_Agent | Context preservation across sessions |
| **Backend** | Python_Requirement_Agent | OpenAPI contract authoring |
| | Python_Dev_Lead | FastAPI implementation |
| | Python_Code_Review | Architecture/security/performance gate |
| | Python_QA | Unit/contract/load/security testing |
| | Python_Commit_Agent | Git commit enforcement |
| **Frontend** | Flutter_Requirement_Agent | Contract -> Dart model generation |
| | Flutter_Dev_Lead | Flutter feature implementation |
| | Flutter_Code_Review | Architecture/UI/accessibility gate |
| | Flutter_QA | Widget/integration/golden testing |
| | Flutter_Commit_Agent | Git commit enforcement |

---

## How to Use This

1. **Open `agent_architecture.html`** in a browser for the interactive visual
2. **Open `flow_diagram.mmd`** in any Mermaid renderer to see the full orchestration flow
3. **Read each agent's `config.json`** to understand its role, skills, and pipeline rules
4. **Read `thought_log.md`** files to see decision rationale from each agent
5. **Check `skills/README.md`** at each level to see the skill catalog
6. **Review `communication_schema.json`** for the inter-agent message protocol

---

## Tech Stack (for reference)

| Layer | Technology |
|-------|-----------|
| Frontend | Flutter, Dart, BLoC, GoRouter, Dio, Freezed, GetIt |
| Backend | FastAPI, SQLAlchemy 2.0, Alembic, Celery, Redis |
| AI | OpenAI GPT-4, LangChain, Qdrant |
| Database | PostgreSQL (async), Redis |
| Cloud | Docker, Cloud Build, Cloud Run, GCS |
| Contract | OpenAPI 3.0.3 (single source of truth) |
| Communication | Redis PubSub (26 message types) |
| State | LangGraph with PostgreSQL checkpointing |
