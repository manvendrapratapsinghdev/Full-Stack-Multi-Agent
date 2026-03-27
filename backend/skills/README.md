# Backend Stack Skills

Skills used by the Python/FastAPI backend agents.

---

## Python Requirement Agent Skills

| Skill | Description | Tools |
|-------|-------------|-------|
| `openapi_authoring` | Write and maintain OpenAPI 3.0.3 spec | FastAPI native generation |
| `pydantic_modeling` | Define Pydantic v2 request/response models | pydantic v2 |
| `spec_linting` | Validate spec quality and consistency | Spectral |
| `breaking_change_detection` | Detect breaking API changes | openapi_diff |
| `contract_testing` | Verify implementation matches contract | Schemathesis |
| `semantic_versioning` | Version the API contract properly | semver |

## Python Dev Lead Skills

| Skill | Description | Tools |
|-------|-------------|-------|
| `fastapi_implementation` | Async endpoint implementation | FastAPI, Uvicorn |
| `database_modeling` | SQLAlchemy 2.0 async models + migrations | SQLAlchemy, Alembic, asyncpg |
| `auth_implementation` | JWT auth with refresh tokens | FastAPI-Users, python-jose |
| `task_queue` | Background job processing | Celery, Redis |
| `observability` | Structured logging + tracing | structlog, OpenTelemetry |
| `cloud_deployment` | Container build + deploy pipeline | Docker, Cloud Build, Cloud Run |
| `caching` | Response and query caching | Redis |
| `image_handling` | Image upload, resize, CDN storage | Pillow, GCS |

## Python Code Review Skills

| Skill | Description | Tools |
|-------|-------------|-------|
| `architecture_review` | Check layer boundaries, DI, circular imports | import_linter, pydeps |
| `security_review` | SQL injection, hardcoded secrets, OWASP Top 10 | Bandit, Semgrep |
| `performance_review` | N+1 queries, missing indexes, async correctness | Manual + profiling |
| `quality_review` | Type hints, error handling, DRY, docstrings | ruff, mypy |
| `migration_review` | Verify Alembic migrations are safe and reversible | Alembic checks |

## Python QA Skills

| Skill | Description | Tools |
|-------|-------------|-------|
| `unit_testing` | Service-level unit tests | pytest |
| `integration_testing` | Full API endpoint testing with real DB | pytest + httpx AsyncClient |
| `contract_testing` | Auto-generate tests from OpenAPI spec | Schemathesis |
| `load_testing` | Performance under load (< 200ms p95, 100K users) | Locust |
| `security_scanning` | Vulnerability detection | Bandit, Safety |
| `mutation_testing` | Test quality verification | mutmut |
| `coverage_tracking` | 85%+ business logic coverage | pytest-cov |

**CI Pipeline:** `ruff check -> ruff format -> mypy -> pytest -> schemathesis -> coverage -> bandit`

## Python Commit Agent Skills

| Skill | Description | Tools |
|-------|-------------|-------|
| `conventional_commits` | Enforce feat/fix/refactor/test prefixes | commitizen |
| `pre_commit_pipeline` | Run linters + tests before commit | pre-commit framework |
| `branch_management` | Feature branch creation and PR | GitHub CLI (gh) |
| `changelog_generation` | Auto-generate CHANGELOG from commits | commitizen |
| `dependency_locking` | Lock Python dependencies | uv.lock |

**Gate Rule:** NO COMMIT without BOTH Code Review PASS AND QA PASS.

**Pre-commit:** `ruff check --fix -> ruff format -> mypy -> pytest (subset)`
