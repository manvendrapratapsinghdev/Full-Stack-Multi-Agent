# Backend Task List - Mai Bhi Editor

**Extracted from:** Sprint_Task_List.md v2.0
**Agent:** Task_Breaking_Agent -> Python_Requirement_Agent
**Date:** 2026-03-27
**Stack:** Backend (Python, FastAPI, SQLAlchemy, PostgreSQL, Redis)
**Total Tasks:** 50 dev tasks + QA tasks

---

## Sprint 0: Foundation

| ID | Title | TRD Ref | Feature | Dependencies | Complexity | Acceptance Criteria |
|----|-------|---------|---------|-------------|------------|---------------------|
| S0-02 | Scaffold FastAPI project with SQLAlchemy + Alembic | -- | -- | None | M | - FastAPI app with health check - SQLAlchemy configured - Alembic migrations ready |
| S0-03 | Define OpenAPI contract v1.0 (all 32 TRD endpoints) | -- | -- | None | L | - contract.yaml covers all endpoints - Validated with spec validator |
| S0-05 | Setup PostgreSQL schema + initial migrations | -- | -- | S0-02 | M | - Initial migration: users, submissions, stories - Forward/backward clean |
| S0-06 | Setup CI/CD pipeline (lint, test, build) | -- | -- | S0-02 | M | - CI on every push: lint + tests - Build artifacts generated |
| S0-07 | Setup Redis cache and object storage connections | -- | -- | S0-02 | S | - Redis pool configured - Object storage client ready |

---

## Sprint 1: Authentication

| ID | Title | TRD Ref | Feature | Dependencies | Complexity | Acceptance Criteria |
|----|-------|---------|---------|-------------|------------|---------------------|
| S1-01 | HT SSO login API endpoint | TRD-AUTH-001 | F-001 | S0-02, S0-05 | M | - POST /api/v1/auth/login - HT SSO validation - Returns JWT pair - 401 on invalid creds |
| S1-02 | JWT token issuance, refresh, and revocation | TRD-AUTH-002 | F-001 | S1-01 | M | - Access TTL 15min, refresh TTL 30d - Refresh endpoint - Logout invalidates token |
| S1-03 | User profile API with creator level | TRD-AUTH-003 | F-002 | S1-01 | M | - GET /api/v1/users/me - creator_level field - Defaults to basic_creator |
| S1-04 | Creator level upgrade logic | TRD-AUTH-003 | F-002 | S1-03 | M | - KYC approval -> ht_approved - Milestone -> trusted_reporter - Audit logged |
| S1-05 | KYC document upload and review queue API | TRD-AUTH-005 | F-022 | S1-03 | L | - POST /api/v1/kyc/upload - AES-256 encryption - KYC review queue for admin |
| S1-10 | Auth API integration tests | TRD-AUTH-001, 002 | F-001 | S1-02 | M | - Login/refresh/logout tested - Token validation tested |
| S1-12 | KYC API integration tests | TRD-AUTH-005 | F-022 | S1-05 | S | - Upload validation tested - Encryption verified |

---

## Sprint 2: Submission

| ID | Title | TRD Ref | Feature | Dependencies | Complexity | Acceptance Criteria |
|----|-------|---------|---------|-------------|------------|---------------------|
| S2-01 | Submission CRUD API | TRD-SUB-001 | F-003 | S1-02 | L | - POST/GET/LIST endpoints - Mandatory field validation - Returns submission_id |
| S2-02 | Image upload with constraints | TRD-SUB-005 | F-027 | S0-07, S2-01 | M | - Max 10MB, JPEG/PNG/HEIC, min 320x240 - 3 compression variants - Max 10 images |
| S2-03 | Submission status lifecycle state machine | TRD-SUB-004 | F-003, F-019 | S2-01 | M | - All state transitions - Timestamps and actor logging - Invalid transitions rejected |
| S2-04 | Multi-image upload endpoint | TRD-SUB-003 | F-004 | S2-02 | S | - Batch upload up to 9 additional - Position field for ordering |
| S2-09 | Submission pipeline integration tests | TRD-SUB-001, 003, 004, 005 | F-003, F-004, F-027 | S2-04 | M | - CRUD tested - Image constraints tested - State transitions tested |

---

## Sprint 3: AI Processing

| ID | Title | TRD Ref | Feature | Dependencies | Complexity | Acceptance Criteria |
|----|-------|---------|---------|-------------|------------|---------------------|
| S3-01 | Language detection service | TRD-AI-001, TRD-AI-008 | F-005, F-038 | S2-01 | M | - Detects Hindi, English, Marathi, Tamil, Telugu, Bengali, Kannada, Gujarati - Returns language_code |
| S3-02 | AI text rephrasing service | TRD-AI-001 | F-005 | S3-01 | L | - LLM integration for editorial rephrase - Same language output - Similarity > 0.85 - Fallback on failure |
| S3-03 | Multilingual rephrase model selection | TRD-AI-008 | F-038 | S3-02 | L | - Language-specific model routing - Quality benchmarks per language - Unsupported language flag |
| S3-04 | AI safety guardrails - text scanning | TRD-AI-003 | F-006 | S2-01 | L | - Hate speech, toxicity, violence, spam detection - Safety score 0.0-1.0 - Auto-reject below 0.2 |
| S3-05 | AI safety guardrails - image scanning | TRD-AI-004 | F-006 | S2-02 | M | - Explicit content, violence, manipulation detection - Contributes to composite score |
| S3-06 | AI tag and category suggestion | TRD-AI-005 | F-007 | S2-01 | S | - Min 3 tags suggested - Category from predefined list - City untouched |
| S3-07 | AI duplicate detection service | TRD-AI-006 | F-008 | S2-01 | L | - Embedding similarity + geo + temporal - Threshold 0.8 - Duplicate clusters |
| S3-08 | AI editorial confidence score calculator | TRD-AI-007 | F-009 | S3-02, S3-04, S3-07 | M | - Weighted formula: safety(0.3) + language(0.2) + duplicate(0.1) + reputation(0.2) + confirmations(0.2) |
| S3-09 | AI orchestration pipeline (async) | TRD-AI-001-007 | F-005-F-009 | S3-08 | L | - Async pipeline with circuit breakers - Status transitions - Error handling |
| S3-12 | AI processing integration tests | TRD-AI-001-008 | F-005-F-009, F-038 | S3-09 | L | - All AI services tested - Quality benchmarks - Fallback scenarios |

---

## Sprint 4: Editorial

| ID | Title | TRD Ref | Feature | Dependencies | Complexity | Acceptance Criteria |
|----|-------|---------|---------|-------------|------------|---------------------|
| S4-01 | Editor review queue API | TRD-ED-001 | F-010 | S3-09 | M | - GET /api/v1/editor/queue - Confidence sort, city filter, pagination |
| S4-02 | Editor action API | TRD-ED-003 | F-011 | S4-01 | M | - Approve/edit/reject/correct actions - Audit logging - Notification triggers |
| S4-03 | False news retroactive flagging API | TRD-ED-004 | F-012 | S4-02 | M | - Flag as false, hide from feed - Points deduction - Demotion trigger |
| S4-04 | Appeal process API | TRD-ED-005 | F-023 | S4-02 | M | - Appeal with justification - Different editor routing - One per submission |
| S4-09 | Editorial workflow integration tests | TRD-ED-001-005 | F-010-F-012, F-023 | S4-04 | M | - All editorial flows tested |

---

## Sprint 5: Feed

| ID | Title | TRD Ref | Feature | Dependencies | Complexity | Acceptance Criteria |
|----|-------|---------|---------|-------------|------------|---------------------|
| S5-01 | News feed API with cursor pagination | TRD-FEED-001 | F-013 | S4-02 | M | - Published stories, recency sort - City filter, cursor pagination - p95 < 200ms |
| S5-02 | Story detail API | TRD-FEED-004 | F-014 | S5-01 | S | - Full story with metadata - 404 for non-existent |
| S5-03 | Story search API | TRD-FEED-006 | F-025 | S5-01 | M | - Full-text search via PostgreSQL FTS - Relevance ranking |
| S5-04 | Location preference API | TRD-FEED-007 | F-036 | S1-03 | S | - GET/PUT preferences - Default city integration |
| S5-05 | Story sharing deep link generation | TRD-FEED-008 | F-035 | S5-02 | S | - Deep link + preview metadata |
| S5-12 | Feed API performance tests | TRD-FEED-001 | F-013 | S5-01 | M | - Load test, p95 < 200ms |

---

## Sprint 6: Community and Reputation

| ID | Title | TRD Ref | Feature | Dependencies | Complexity | Acceptance Criteria |
|----|-------|---------|---------|-------------|------------|---------------------|
| S6-01 | Confirm story API | TRD-COM-001 | F-015 | S5-01 | S | - One per user per story - Increments count |
| S6-02 | Like story API (toggle) | TRD-COM-002 | F-016 | S5-01 | S | - Toggle behavior - Returns updated count |
| S6-03 | Reputation points engine | TRD-REP-001 | F-017 | S4-02, S4-03 | M | - Points: publish+10, confirm+2, like+1 - Deduct on false - Milestones - Atomic ops |
| S6-04 | Creator public profile API | TRD-REP-002 | F-018 | S6-03 | M | - Profile + story list - Own profile includes points |
| S6-05 | Block and report APIs | TRD-COM-003 | F-026 | S5-01, S6-04 | M | - Report/block/unblock/list - Feed excludes blocked |
| S6-10 | Community and reputation integration tests | TRD-COM-001-003, TRD-REP-001-002 | F-015-F-018, F-026 | S6-05 | M | - All community APIs tested |

---

## Sprint 7: Notifications

| ID | Title | TRD Ref | Feature | Dependencies | Complexity | Acceptance Criteria |
|----|-------|---------|---------|-------------|------------|---------------------|
| S7-01 | FCM push notification service | TRD-NOTIF-001 | F-021 | S4-02, S6-03 | L | - FCM integration - All triggers - < 60s delivery - Logging |
| S7-02 | Notification preferences API | TRD-NOTIF-002 | F-021 | S7-01 | S | - GET/PUT per category |
| S7-03 | FCM device token registration | TRD-NOTIF-001 | F-021 | S7-01 | S | - Token registration - Multi-device |
| S7-04 | i18n string externalization (backend) | TRD-CROSS-002 | F-031 | None | M | - Notification templates Hindi+English |
| S7-09 | Notification service integration tests | TRD-NOTIF-001, 002 | F-021 | S7-03 | M | - FCM mock tested - Timing, preferences, batching |

---

## Sprint 8: Safety and Privacy

| ID | Title | TRD Ref | Feature | Dependencies | Complexity | Acceptance Criteria |
|----|-------|---------|---------|-------------|------------|---------------------|
| S8-01 | Rate limiting middleware | TRD-PLAT-005 | F-033 | S1-02, S2-01 | M | - Per-user and per-IP limits - HTTP 429 - Confirmation spam prevention |
| S8-02 | Bot detection and account flagging | TRD-PLAT-005 | F-033 | S8-01 | M | - Pattern detection - Flagging - Temp restriction |
| S8-03 | Account deletion API | TRD-PLAT-004 | F-032 | S1-03 | M | - 30-day grace - Anonymization - KYC deletion |
| S8-04 | Consent management API | TRD-PLAT-004 | F-032 | S1-01 | S | - Consent record/retrieve/revoke |
| S8-05 | Audit trail for data access | TRD-PLAT-004 | F-032 | S1-02 | M | - Immutable append-only log - KYC access tracking |
| S8-06 | Analytics event tracking infrastructure | TRD-PLAT-003 | F-029 | S2-01, S5-01 | L | - Event ingestion API - Schema validation - Funnel tracking |
| S8-10 | Privacy and rate limiting integration tests | TRD-PLAT-004, 005 | F-032, F-033 | S8-05 | M | - Rate limits, deletion, consent, audit tested |
| S8-11 | Analytics event tests | TRD-PLAT-003 | F-029 | S8-06 | S | - Ingestion, schema, funnel tested |

---

## Sprint 9: Admin and Advanced

| ID | Title | TRD Ref | Feature | Dependencies | Complexity | Acceptance Criteria |
|----|-------|---------|---------|-------------|------------|---------------------|
| S9-01 | Admin moderation panel API | TRD-PLAT-001 | F-037 | S1-05, S4-04, S6-05 | L | - All moderation queues - Warn/suspend/ban - Config management - Audit logged |
| S9-02 | Editor dashboard analytics API | TRD-PLAT-002 | F-034 | S8-06 | M | - Metrics: pending, approval time, rejection rate, published - City filter - AI accuracy |
| S9-03 | Trending stories algorithm | TRD-FEED-005 | F-020 | S6-01, S6-02 | M | - Weighted scoring - Per-city - Threshold gating |
| S9-08 | Admin panel integration tests | TRD-PLAT-001 | F-037 | S9-01 | M | - All queues and actions tested |
| S9-09 | Trending algorithm tests | TRD-FEED-005 | F-020 | S9-03 | S | - Scoring, city scope, threshold tested |
| S9-10 | End-to-end integration testing (full flow) | All TRDs | All | S9-07 | L | - Full cross-stack flow tested |

---

## Total: 50 Backend tasks (38 dev + 12 QA)

---

## API Endpoints Summary (for contract.yaml)

| Method | Endpoint | Sprint | TRD Ref |
|--------|----------|--------|---------|
| POST | /api/v1/auth/login | S1 | TRD-AUTH-001 |
| POST | /api/v1/auth/refresh | S1 | TRD-AUTH-002 |
| POST | /api/v1/auth/logout | S1 | TRD-AUTH-002 |
| GET | /api/v1/users/me | S1 | TRD-AUTH-003 |
| PUT | /api/v1/users/me/preferences | S5 | TRD-FEED-007 |
| GET | /api/v1/users/me/preferences | S5 | TRD-FEED-007 |
| GET | /api/v1/users/me/consent | S8 | TRD-PLAT-004 |
| PUT | /api/v1/users/me/consent | S8 | TRD-PLAT-004 |
| DELETE | /api/v1/users/me | S8 | TRD-PLAT-004 |
| POST | /api/v1/kyc/upload | S1 | TRD-AUTH-005 |
| POST | /api/v1/submissions | S2 | TRD-SUB-001 |
| GET | /api/v1/submissions/{id} | S2 | TRD-SUB-001 |
| GET | /api/v1/submissions | S2 | TRD-SUB-001 |
| POST | /api/v1/submissions/{id}/images | S2 | TRD-SUB-003 |
| POST | /api/v1/submissions/{id}/appeal | S4 | TRD-ED-005 |
| GET | /api/v1/editor/queue | S4 | TRD-ED-001 |
| POST | /api/v1/editor/submissions/{id}/action | S4 | TRD-ED-003 |
| POST | /api/v1/editor/stories/{id}/flag-false | S4 | TRD-ED-004 |
| GET | /api/v1/editor/dashboard | S9 | TRD-PLAT-002 |
| GET | /api/v1/feed | S5 | TRD-FEED-001 |
| GET | /api/v1/feed/search | S5 | TRD-FEED-006 |
| GET | /api/v1/stories/{id} | S5 | TRD-FEED-004 |
| GET | /api/v1/stories/{id}/share-link | S5 | TRD-FEED-008 |
| POST | /api/v1/stories/{id}/confirm | S6 | TRD-COM-001 |
| POST | /api/v1/stories/{id}/like | S6 | TRD-COM-002 |
| GET | /api/v1/creators/{id}/profile | S6 | TRD-REP-002 |
| POST | /api/v1/reports | S6 | TRD-COM-003 |
| POST | /api/v1/blocks/{creator_id} | S6 | TRD-COM-003 |
| DELETE | /api/v1/blocks/{creator_id} | S6 | TRD-COM-003 |
| GET | /api/v1/blocks | S6 | TRD-COM-003 |
| POST | /api/v1/devices/register | S7 | TRD-NOTIF-001 |
| GET | /api/v1/users/me/notification-preferences | S7 | TRD-NOTIF-002 |
| PUT | /api/v1/users/me/notification-preferences | S7 | TRD-NOTIF-002 |
| POST | /api/v1/events | S8 | TRD-PLAT-003 |
| GET | /api/v1/admin/moderation/reports | S9 | TRD-PLAT-001 |
| GET | /api/v1/admin/moderation/kyc | S9 | TRD-PLAT-001 |
| GET | /api/v1/admin/moderation/appeals | S9 | TRD-PLAT-001 |
| POST | /api/v1/admin/creators/{id}/action | S9 | TRD-PLAT-001 |
| PUT | /api/v1/admin/config | S9 | TRD-PLAT-001 |

---

**END OF BACKEND TASK LIST**
