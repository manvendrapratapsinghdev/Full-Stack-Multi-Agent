# Sprint Task List - Mai Bhi Editor

**Generated from:** Master_TRD v2.0.0
**Agent:** Task_Breaking_Agent
**Date:** 2026-03-27
**Total Sprints:** 10 (S0-S9)
**Total Tasks:** 138

---

## Complexity Scale

| Size | Effort | Description |
|------|--------|-------------|
| S | < 2 hours | Single file change |
| M | 2-6 hours | Multiple files, one module |
| L | 6-16 hours | Multiple modules, integration required |

---

## Sprint 0: Foundation and Project Setup

**Goal:** Scaffold both projects, define API contract, setup CI/CD
**Duration:** 1 week

| ID | Title | Stack | Agent | TRD Ref | Feature Ref | Dependencies | Complexity | Acceptance Criteria |
|----|-------|-------|-------|---------|-------------|-------------|------------|---------------------|
| S0-01 | Scaffold Flutter project with Clean Architecture (BLoC, layers) | Frontend | Flutter_Dev_Lead | -- | -- | None | M | - Project structure with data/domain/presentation layers created - BLoC dependency injection configured - Base app runs on iOS and Android simulators |
| S0-02 | Scaffold FastAPI project with SQLAlchemy + Alembic | Backend | Python_Dev_Lead | -- | -- | None | M | - FastAPI app starts with health check endpoint - SQLAlchemy models base configured - Alembic migration setup complete |
| S0-03 | Define OpenAPI contract v1.0 (all 32 TRD endpoints) | Backend | Python_Requirement_Agent | -- | -- | None | L | - contract.yaml covers all endpoints from TRD specs - Validated with openapi-spec-validator - Shared to Flutter team |
| S0-04 | Generate Flutter data models from OpenAPI contract | Frontend | Flutter_Requirement_Agent | -- | -- | S0-03 | M | - All request/response models generated via build_runner - Models compile without errors - JSON serialization tests pass |
| S0-05 | Setup PostgreSQL schema + initial migrations | Backend | Python_Dev_Lead | -- | -- | S0-02 | M | - Initial migration creates users, submissions, stories tables - Migration runs forward and backward cleanly |
| S0-06 | Setup CI/CD pipeline (lint, test, build) | Backend | Python_Dev_Lead | -- | -- | S0-02 | M | - CI runs on every push: lint + unit tests - Flutter CI: analyze + test - Build artifacts generated |
| S0-07 | Setup Redis cache and object storage connections | Backend | Python_Dev_Lead | -- | -- | S0-02 | S | - Redis connection pool configured - Object storage client configured with test bucket |
| S0-08 | Setup Flutter theme, design tokens, and accessibility base | Frontend | Flutter_Dev_Lead | TRD-CROSS-001 | F-030 | S0-01 | M | - Color palette with 4.5:1 contrast ratios defined - Semantics widgets in base components - Theme supports light mode |

**Parallel Groups:**
- Group A: S0-01, S0-02, S0-03 (all independent)
- Group B: S0-04, S0-05, S0-06, S0-07 (depend on Group A)
- Group C: S0-08 (depends on S0-01)

---

## Sprint 1: Authentication and User Identity

**Goal:** HT SSO login, JWT tokens, creator levels, KYC flow backend
**Duration:** 2 weeks
**Critical Path:** F-001 -> F-002 (blocks everything downstream)

| ID | Title | Stack | Agent | TRD Ref | Feature Ref | Dependencies | Complexity | Acceptance Criteria |
|----|-------|-------|-------|---------|-------------|-------------|------------|---------------------|
| S1-01 | HT SSO login API endpoint (POST /api/v1/auth/login) | Backend | Python_Dev_Lead | TRD-AUTH-001 | F-001 | S0-02, S0-05 | M | - Endpoint validates credentials against HT SSO mock - Returns JWT access + refresh token pair - Invalid creds return 401 |
| S1-02 | JWT token issuance, refresh, and revocation | Backend | Python_Dev_Lead | TRD-AUTH-002 | F-001 | S1-01 | M | - Access token TTL 15 min, refresh TTL 30 days - POST /api/v1/auth/refresh works - POST /api/v1/auth/logout invalidates refresh token |
| S1-03 | User profile API with creator level field | Backend | Python_Dev_Lead | TRD-AUTH-003 | F-002 | S1-01 | M | - GET /api/v1/users/me returns profile with creator_level - New users default to "basic_creator" - Level field is read-only via this endpoint |
| S1-04 | Creator level upgrade logic (KYC and milestone triggers) | Backend | Python_Dev_Lead | TRD-AUTH-003 | F-002 | S1-03 | M | - Level upgrades from basic->approved on KYC approval event - Level upgrades from approved->trusted on milestone threshold - Level changes are audit-logged |
| S1-05 | KYC document upload and review queue API | Backend | Python_Dev_Lead | TRD-AUTH-005 | F-022 | S1-03 | L | - POST /api/v1/kyc/upload accepts document (JPEG/PNG/PDF, max 5MB) - Document encrypted at rest (AES-256) - KYC entry created with status "pending" - GET /api/v1/admin/kyc/queue returns pending reviews |
| S1-06 | Login screen UI (BLoC pattern) | Frontend | Flutter_Dev_Lead | TRD-AUTH-001 | F-001 | S0-01, S0-04, S1-01 | M | - Login form with email/phone + password fields - Validation on empty fields - Successful login navigates to home - Error states displayed |
| S1-07 | Auth state management + secure token storage | Frontend | Flutter_Dev_Lead | TRD-AUTH-002 | F-001 | S1-06 | M | - Tokens stored in flutter_secure_storage - HTTP interceptor auto-refreshes expired access tokens - Logout clears all local state |
| S1-08 | Profile screen with creator badge display | Frontend | Flutter_Dev_Lead | TRD-AUTH-004 | F-002 | S1-07 | M | - Profile screen shows name, avatar, level badge - Badge renders correct icon/color per level - Own profile shows additional stats |
| S1-09 | KYC upload screen UI | Frontend | Flutter_Dev_Lead | TRD-AUTH-005 | F-022 | S1-08 | M | - Document upload form with accepted ID types listed - Camera/gallery picker for document image - Upload progress indicator - Success/error states |
| S1-10 | Auth API integration tests | Backend | Python_QA | TRD-AUTH-001, 002 | F-001 | S1-02 | M | - Login success/failure scenarios tested - Token refresh tested - Logout invalidation tested |
| S1-11 | Auth UI widget tests | Frontend | Flutter_QA | TRD-AUTH-001 | F-001 | S1-07 | S | - Login form validation tested - Error states render correctly - Navigation on success tested |
| S1-12 | KYC API integration tests | Backend | Python_QA | TRD-AUTH-005 | F-022 | S1-05 | S | - Upload validation tested (format, size) - Encryption at rest verified - Queue retrieval tested |

**Parallel Groups:**
- Group A (Backend): S1-01 -> S1-02 -> S1-03 -> S1-04 -> S1-05 (sequential)
- Group B (Frontend): S1-06 -> S1-07 -> S1-08 -> S1-09 (sequential, starts after S1-01 available)
- Group C (QA): S1-10, S1-11, S1-12 (after dev tasks complete)

**Cross-Stack Handshake:** S1-01 (Backend API) <-> S1-06 (Frontend integration) -- contract alignment required

---

## Sprint 2: Submission Pipeline

**Goal:** News submission form, image upload, media constraints, dashboard, offline drafts
**Duration:** 2 weeks
**Critical Path:** F-003 (blocks entire AI tier)

| ID | Title | Stack | Agent | TRD Ref | Feature Ref | Dependencies | Complexity | Acceptance Criteria |
|----|-------|-------|-------|---------|-------------|-------------|------------|---------------------|
| S2-01 | Submission CRUD API (POST, GET, LIST) | Backend | Python_Dev_Lead | TRD-SUB-001 | F-003 | S1-02 | L | - POST /api/v1/submissions creates submission with mandatory fields - GET by ID returns submission with status - LIST returns creator's submissions with filtering |
| S2-02 | Image upload to object storage with constraints | Backend | Python_Dev_Lead | TRD-SUB-005 | F-027 | S0-07, S2-01 | M | - Upload validates: max 10MB, JPEG/PNG/HEIC, min 320x240 - Server-side compression generates 3 variants (thumb/feed/full) - Max 10 images per submission enforced |
| S2-03 | Submission status lifecycle state machine | Backend | Python_Dev_Lead | TRD-SUB-004 | F-003, F-019 | S2-01 | M | - State transitions: draft->in_progress->ai_processing->under_review->published/rejected/correction_needed - Each transition logged with timestamp and actor - Invalid transitions rejected |
| S2-04 | Multi-image upload endpoint | Backend | Python_Dev_Lead | TRD-SUB-003 | F-004 | S2-02 | S | - Endpoint accepts batch image upload (up to 9 additional) - Returns array of image URLs - Reorder support via position field |
| S2-05 | News submission form UI (title, image, desc, city) | Frontend | Flutter_Dev_Lead | TRD-SUB-002 | F-003 | S0-04, S1-07, S2-01 | L | - Form with all mandatory fields - Inline validation on each field - City searchable dropdown from API - Submit button disabled until valid - Form state preserved on background |
| S2-06 | Multi-image picker and upload UI | Frontend | Flutter_Dev_Lead | TRD-SUB-003 | F-004 | S2-05 | M | - "Add More Images" button opens gallery/camera picker - Thumbnails with drag-reorder and delete - Client-side size/format validation before upload - Upload progress per image |
| S2-07 | "Posted by You" dashboard UI | Frontend | Flutter_Dev_Lead | TRD-SUB-006 | F-019 | S2-05 | M | - Lists creator's submissions sorted by recency - Status filter chips (Draft, In Progress, Under Review, Published, Rejected) - Rejection reason visible on rejected items - Empty state with CTA |
| S2-08 | Offline draft support (local persistence) | Frontend | Flutter_Dev_Lead | TRD-SUB-007 | F-028 | S2-05, S2-07 | M | - Auto-save form state to Hive/SQLite on field changes - Drafts appear in dashboard under "Draft" status - Tap draft restores all fields including images - Multiple drafts supported |
| S2-09 | Submission pipeline integration tests | Backend | Python_QA | TRD-SUB-001, 003, 004, 005 | F-003, F-004, F-027 | S2-04 | M | - CRUD operations tested - Image constraints validated - Status transitions tested - Edge cases: oversized images, invalid formats |
| S2-10 | Submission form widget tests | Frontend | Flutter_QA | TRD-SUB-002 | F-003 | S2-05 | S | - Form validation tested - City dropdown tested - Background preservation tested |

**Parallel Groups:**
- Group A (Backend): S2-01 -> S2-02 -> S2-03 -> S2-04 (sequential)
- Group B (Frontend): S2-05 -> S2-06 -> S2-07 -> S2-08 (sequential, starts after S2-01)
- Group C (QA): S2-09, S2-10 (after dev)

**Cross-Stack Handshake:** S2-01 (Backend API) <-> S2-05 (Frontend form) -- contract alignment required

---

## Sprint 3: AI Processing Pipeline

**Goal:** Language rephrasing, safety guardrails, tag suggestion, duplicate detection, confidence score
**Duration:** 2 weeks
**Critical Path:** F-005 -> F-009 (confidence score depends on rephrase + safety)

| ID | Title | Stack | Agent | TRD Ref | Feature Ref | Dependencies | Complexity | Acceptance Criteria |
|----|-------|-------|-------|---------|-------------|-------------|------------|---------------------|
| S3-01 | Language detection service | Backend | Python_Dev_Lead | TRD-AI-001, TRD-AI-008 | F-005, F-038 | S2-01 | M | - Auto-detects language from submission text - Supports Hindi, English, Marathi, Tamil, Telugu, Bengali, Kannada, Gujarati - Returns language_code metadata |
| S3-02 | AI text rephrasing service (LLM integration) | Backend | Python_Dev_Lead | TRD-AI-001 | F-005 | S3-01 | L | - Rephrases text to editorial quality in same language as input - Semantic similarity > 0.85 with original - Stores both original and rephrased text - Fallback: original text proceeds if AI fails |
| S3-03 | Multilingual rephrase model selection | Backend | Python_Dev_Lead | TRD-AI-008 | F-038 | S3-02 | L | - Routes to appropriate language model based on detected language - Quality benchmarks per language - Unsupported language flag: "ai_rephrase_unavailable" |
| S3-04 | AI safety guardrails - text scanning | Backend | Python_Dev_Lead | TRD-AI-003 | F-006 | S2-01 | L | - Scans for hate speech, toxicity, violence, spam - Returns safety_score (0.0-1.0) - Score < 0.2: auto-reject with reason - Score 0.2-0.6: proceed with warning - Score > 0.6: pass |
| S3-05 | AI safety guardrails - image scanning | Backend | Python_Dev_Lead | TRD-AI-004 | F-006 | S2-02 | M | - Scans images for explicit content, violence, manipulation - Image safety scores contribute to composite safety score - Explicit content: block submission |
| S3-06 | AI tag and category suggestion service | Backend | Python_Dev_Lead | TRD-AI-005 | F-007 | S2-01 | S | - Analyzes content and suggests min 3 tags - Suggests category from predefined list - City field never modified |
| S3-07 | AI duplicate detection service | Backend | Python_Dev_Lead | TRD-AI-006 | F-008 | S2-01 | L | - Embedding-based similarity with existing stories - Combined with geo (city) and temporal (24h) proximity - Similarity > 0.8 triggers duplicate alert - Groups duplicates into clusters for editors |
| S3-08 | AI editorial confidence score calculator | Backend | Python_Dev_Lead | TRD-AI-007 | F-009 | S3-02, S3-04, S3-07 | M | - Composite score: safety(0.3) + language(0.2) + duplicate(0.1) + reputation(0.2) + confirmations(0.2) - Score stored with submission - Determines default queue ordering |
| S3-09 | AI orchestration pipeline (async processing) | Backend | Python_Dev_Lead | TRD-AI-001-007 | F-005, F-006, F-007, F-008, F-009 | S3-08 | L | - Async pipeline: rephrase -> safety -> tags -> duplicates -> confidence score - Circuit breaker per service - Status transitions: in_progress -> ai_processing -> under_review - Error handling: partial failure proceeds with available results |
| S3-10 | "Your post corrected by AI" preview screen | Frontend | Flutter_Dev_Lead | TRD-AI-002 | F-005 | S0-04, S2-05, S3-02 | M | - Shows original text and AI-rephrased version side-by-side - Header: "Your post -- corrected by AI" - "Proceed" button to continue - No edit capability on AI text |
| S3-11 | AI tag suggestion UI (selectable chips) | Frontend | Flutter_Dev_Lead | TRD-AI-005 | F-007 | S3-10 | S | - AI-suggested tags displayed as chips - Deselect to remove tag - Category dropdown with AI pre-selection - City remains user-controlled |
| S3-12 | AI processing integration tests | Backend | Python_QA | TRD-AI-001-008 | F-005, F-006, F-007, F-008, F-009, F-038 | S3-09 | L | - Rephrase quality tested per language - Safety scoring tested with known inputs - Duplicate detection accuracy tested - Confidence score formula verified - Pipeline fallbacks tested |

**Parallel Groups:**
- Group A: S3-01 -> S3-02 -> S3-03 (language chain)
- Group B: S3-04, S3-05 (safety -- parallel to Group A after S2-01)
- Group C: S3-06, S3-07 (tags + duplicates -- parallel to Group A after S2-01)
- Group D: S3-08 -> S3-09 (depends on A, B, C)
- Group E (Frontend): S3-10 -> S3-11 (starts after S3-02 available)
- Group F (QA): S3-12 (after all)

**Cross-Stack Handshake:** S3-09 (Backend pipeline status) <-> S3-10 (Frontend preview) -- polling/webhook alignment needed

---

## Sprint 4: Editorial Workflow

**Goal:** Editor review queue, editor actions, false news handling, appeal process
**Duration:** 2 weeks
**Critical Path:** F-010 -> F-011 -> F-013

| ID | Title | Stack | Agent | TRD Ref | Feature Ref | Dependencies | Complexity | Acceptance Criteria |
|----|-------|-------|-------|---------|-------------|-------------|------------|---------------------|
| S4-01 | Editor review queue API (GET with filters and sort) | Backend | Python_Dev_Lead | TRD-ED-001 | F-010 | S3-09 | M | - GET /api/v1/editor/queue returns under_review submissions - Default sort: confidence score desc - City filter support - Pagination with cursor |
| S4-02 | Editor action API (approve/edit/reject/correct) | Backend | Python_Dev_Lead | TRD-ED-003 | F-011 | S4-01 | M | - POST /api/v1/editor/submissions/{id}/action - Approve: status->published, story created in feed - Reject: reason required, status->rejected - Edit: modified content saved, then published - Correct: comments required, status->correction_needed - Audit log entry for every action |
| S4-03 | False news retroactive flagging API | Backend | Python_Dev_Lead | TRD-ED-004 | F-012 | S4-02 | M | - POST /api/v1/editor/stories/{id}/flag-false - Story status->false_report, hidden from feed - Creator points deducted - Warning notification sent - Repeated flags trigger demotion review |
| S4-04 | Appeal process API | Backend | Python_Dev_Lead | TRD-ED-005 | F-023 | S4-02 | M | - POST /api/v1/submissions/{id}/appeal with justification - Routed to different editor than original reviewer - Appeal editor can approve or uphold rejection - One appeal per submission enforced |
| S4-05 | Editor review queue UI | Frontend | Flutter_Dev_Lead | TRD-ED-002 | F-010 | S0-04, S4-01 | L | - Queue screen with submission cards - City filter dropdown, sort toggle - Card shows: title, city, confidence badge, safety indicator, confirmation count, time - Auto-refresh every 30s - Infinite scroll pagination |
| S4-06 | Editor review detail + action UI | Frontend | Flutter_Dev_Lead | TRD-ED-003 | F-011 | S4-05 | L | - Detail view: AI draft, original text, safety score, duplicate alerts, confirmations - Action buttons: Approve, Edit, Reject, Mark Correction - Reject/Correct require text input - Confirmation dialog before irreversible actions |
| S4-07 | False news flag UI (editor story action) | Frontend | Flutter_Dev_Lead | TRD-ED-004 | F-012 | S4-06 | S | - "Flag as False" button on published story detail (editor view) - Confirmation dialog with warning about reputation impact - Success/error feedback |
| S4-08 | Appeal submission UI (creator dashboard) | Frontend | Flutter_Dev_Lead | TRD-ED-005 | F-023 | S2-07, S4-04 | M | - "Appeal" button on rejected submissions in dashboard - Appeal form with justification text area - One appeal per submission (button disabled after appeal) - Appeal status visible in dashboard |
| S4-09 | Editorial workflow integration tests | Backend | Python_QA | TRD-ED-001-005 | F-010, F-011, F-012, F-023 | S4-04 | M | - Queue filtering and sorting tested - All 4 editor actions tested with state transitions - False news flow tested (including reputation deduction) - Appeal routing to different editor verified |
| S4-10 | Editor UI widget tests | Frontend | Flutter_QA | TRD-ED-002, 003 | F-010, F-011 | S4-06 | S | - Queue rendering tested - Action buttons tested - Detail view layout verified |

**Parallel Groups:**
- Group A (Backend): S4-01 -> S4-02 -> S4-03 -> S4-04 (sequential)
- Group B (Frontend): S4-05 -> S4-06 -> S4-07, S4-08 (S4-07 and S4-08 parallel)
- Group C (QA): S4-09, S4-10 (after dev)

**Cross-Stack Handshake:** S4-01/S4-02 (Backend APIs) <-> S4-05/S4-06 (Frontend queue+actions) -- contract alignment critical

---

## Sprint 5: News Feed and Publishing

**Goal:** Feed API, feed UI, story detail, story cards, search, location personalization
**Duration:** 2 weeks
**Critical Path:** F-013 -> F-014 (blocks community features)

| ID | Title | Stack | Agent | TRD Ref | Feature Ref | Dependencies | Complexity | Acceptance Criteria |
|----|-------|-------|-------|---------|-------------|-------------|------------|---------------------|
| S5-01 | News feed API with cursor pagination | Backend | Python_Dev_Lead | TRD-FEED-001 | F-013 | S4-02 | M | - GET /api/v1/feed returns published stories - Default sort: published_at desc - City filter parameter - Cursor pagination (page_size=20) - p95 response < 200ms |
| S5-02 | Story detail API | Backend | Python_Dev_Lead | TRD-FEED-004 | F-014 | S5-01 | S | - GET /api/v1/stories/{id} returns full story data - Includes: content, images[], creator profile, editor name, badges, counts - Returns 404 for non-existent/unpublished stories |
| S5-03 | Story search API (full-text) | Backend | Python_Dev_Lead | TRD-FEED-006 | F-025 | S5-01 | M | - GET /api/v1/feed/search?q= - Full-text search via PostgreSQL FTS - Results ranked by relevance, recency tiebreaker - Supports keyword, city, tag, creator name |
| S5-04 | Location preference API | Backend | Python_Dev_Lead | TRD-FEED-007 | F-036 | S1-03 | S | - PUT /api/v1/users/me/preferences with preferred_city - GET /api/v1/users/me/preferences returns preferences - Feed API uses preferred_city as default filter when no explicit city param |
| S5-05 | Story sharing deep link generation | Backend | Python_Dev_Lead | TRD-FEED-008 | F-035 | S5-02 | S | - GET /api/v1/stories/{id}/share-link returns deep link URI + preview metadata - Deep link format: htapp://stories/{id} - Web fallback URL for non-app users |
| S5-06 | "News By You" feed screen UI | Frontend | Flutter_Dev_Lead | TRD-FEED-002 | F-013 | S0-04, S5-01 | L | - Top-level section parallel to "News For You" - Story cards in scrollable list - City filter at top - Infinite scroll - Pull to refresh - Empty state per city |
| S5-07 | Story card widget | Frontend | Flutter_Dev_Lead | TRD-FEED-003 | F-013 | S5-06 | M | - Displays: title (2-line max), cover image (16:9), creator name+badge, like count, confirmation count, Confirm button, city, relative time - Tapping navigates to detail page |
| S5-08 | Story detail page UI | Frontend | Flutter_Dev_Lead | TRD-FEED-004 | F-014 | S5-07 | M | - Full content, swipeable image gallery - Creator name (tappable -> profile), editor name - AI Verified + Editor Verified badges - Like and Confirm buttons - Share button |
| S5-09 | Search UI within "News By You" | Frontend | Flutter_Dev_Lead | TRD-FEED-006 | F-025 | S5-06 | M | - Search icon in app bar - Search input with real-time suggestions - Results in story card format - Empty state for no results |
| S5-10 | City preference selection + location feed | Frontend | Flutter_Dev_Lead | TRD-FEED-007 | F-036 | S5-06 | S | - First-time city selection prompt - City selector in settings - Optional GPS location for city suggestion - Feed defaults to preferred city |
| S5-11 | Story share button integration | Frontend | Flutter_Dev_Lead | TRD-FEED-008 | F-035 | S5-08 | S | - Share button triggers native OS share sheet - Share payload: title, link, preview snippet - Deep link handling for incoming links |
| S5-12 | Feed API performance tests (p95 < 200ms) | Backend | Python_QA | TRD-FEED-001 | F-013 | S5-01 | M | - Load test with 1000 concurrent requests - p95 response time under 200ms - Pagination correctness under load |
| S5-13 | Feed and detail UI widget tests | Frontend | Flutter_QA | TRD-FEED-002, 003, 004 | F-013, F-014 | S5-08 | S | - Story card renders correctly - Detail page layout verified - Infinite scroll behavior tested |

**Parallel Groups:**
- Group A (Backend): S5-01 -> S5-02 -> S5-03 (sequential), S5-04 and S5-05 parallel to S5-03
- Group B (Frontend): S5-06 -> S5-07 -> S5-08 -> S5-09, S5-10, S5-11 (last three parallel)
- Group C (QA): S5-12, S5-13 (after dev)

**Cross-Stack Handshake:** S5-01/S5-02 (Backend APIs) <-> S5-06/S5-07/S5-08 (Frontend feed+detail) -- contract alignment required

---

## Sprint 6: Community Engagement and Reputation

**Goal:** Confirm, like, reputation engine, creator profiles, blocking/reporting
**Duration:** 2 weeks

| ID | Title | Stack | Agent | TRD Ref | Feature Ref | Dependencies | Complexity | Acceptance Criteria |
|----|-------|-------|-------|---------|-------------|-------------|------------|---------------------|
| S6-01 | Confirm story API | Backend | Python_Dev_Lead | TRD-COM-001 | F-015 | S5-01 | S | - POST /api/v1/stories/{id}/confirm - One per user per story (unique constraint) - Increments confirmation_count - Returns updated count |
| S6-02 | Like story API (toggle) | Backend | Python_Dev_Lead | TRD-COM-002 | F-016 | S5-01 | S | - POST /api/v1/stories/{id}/like (toggle) - Like/unlike behavior - Returns updated like_count |
| S6-03 | Reputation points engine | Backend | Python_Dev_Lead | TRD-REP-001 | F-017 | S4-02, S4-03 | M | - Points awarded on publish event (+10) - Points for confirmations received (+2 each) - Points for likes received (+1 each) - Points deducted on false flag - Milestones: 100=badge, 1000=gifts eligible - Atomic operations prevent race conditions |
| S6-04 | Creator public profile API | Backend | Python_Dev_Lead | TRD-REP-002 | F-018 | S6-03 | M | - GET /api/v1/creators/{id}/profile - Returns: name, avatar, level, badges, published_count, stories[] - Own profile includes reputation_points_total - Stories paginated |
| S6-05 | Block and report APIs | Backend | Python_Dev_Lead | TRD-COM-003 | F-026 | S5-01, S6-04 | M | - POST /api/v1/reports with reason category - POST /api/v1/blocks/{creator_id} - DELETE /api/v1/blocks/{creator_id} (unblock) - GET /api/v1/blocks (block list) - Feed API excludes blocked creators |
| S6-06 | Confirm + like UI interactions | Frontend | Flutter_Dev_Lead | TRD-COM-001, TRD-COM-002 | F-015, F-016 | S5-07, S5-08 | M | - Confirm button on cards and detail page - Optimistic UI update, server sync - Already confirmed: disabled "Confirmed" state - Like toggle with animation - Unauthenticated: prompt login |
| S6-07 | Creator public profile page UI | Frontend | Flutter_Dev_Lead | TRD-REP-003 | F-018 | S6-04 | M | - Tappable creator name navigates to profile - Shows: name, avatar, level badge, badge collection, published count, story list - Own profile shows reputation points - Trusted Reporter badge prominent |
| S6-08 | Block and report UI | Frontend | Flutter_Dev_Lead | TRD-COM-003 | F-026 | S6-07 | M | - Report button on story/profile with reason categories - Block button on profile - Block list in settings with unblock option - Blocked creators filtered from feed |
| S6-09 | Reputation badge display widget | Frontend | Flutter_Dev_Lead | TRD-AUTH-004, TRD-REP-001 | F-002, F-017 | S6-07 | S | - Milestone badge collection rendered - Point total on own profile - Badge icon + color per level - Milestone notification shown on achievement |
| S6-10 | Community and reputation integration tests | Backend | Python_QA | TRD-COM-001-003, TRD-REP-001-002 | F-015, F-016, F-017, F-018, F-026 | S6-05 | M | - Confirm idempotency tested - Like toggle tested - Reputation point calculations verified - Profile API response tested - Block filtering tested - Report queue tested |
| S6-11 | Community UI widget tests | Frontend | Flutter_QA | TRD-COM-001, 002 | F-015, F-016 | S6-06 | S | - Confirm/like button states tested - Optimistic update tested - Profile page layout verified |

**Parallel Groups:**
- Group A (Backend): S6-01 + S6-02 (parallel), S6-03, S6-04, S6-05 (sequential after A)
- Group B (Frontend): S6-06 (after S6-01/S6-02), S6-07 (after S6-04), S6-08 (after S6-05), S6-09 (after S6-07)
- Group C (QA): S6-10, S6-11 (after dev)

**Cross-Stack Handshake:** S6-01/S6-02 (Backend APIs) <-> S6-06 (Frontend buttons) -- optimistic update sync pattern

---

## Sprint 7: Notifications and Onboarding

**Goal:** Push notifications, notification preferences, onboarding flow, i18n setup
**Duration:** 2 weeks

| ID | Title | Stack | Agent | TRD Ref | Feature Ref | Dependencies | Complexity | Acceptance Criteria |
|----|-------|-------|-------|---------|-------------|-------------|------------|---------------------|
| S7-01 | FCM push notification service (backend) | Backend | Python_Dev_Lead | TRD-NOTIF-001 | F-021 | S4-02, S6-03 | L | - FCM integration for sending push notifications - Triggers: story approval, rejection, correction, milestone, community activity (batched) - Delivery within 60 seconds of trigger event - Delivery status logged |
| S7-02 | Notification preferences API | Backend | Python_Dev_Lead | TRD-NOTIF-002 | F-021 | S7-01 | S | - GET/PUT /api/v1/users/me/notification-preferences - Categories: story_status, milestones, community_activity, trending - Server-side storage, synced across devices |
| S7-03 | FCM device token registration | Backend | Python_Dev_Lead | TRD-NOTIF-001 | F-021 | S7-01 | S | - POST /api/v1/devices/register with FCM token - Supports multiple devices per user - Token refresh handling |
| S7-04 | i18n string externalization (backend) | Backend | Python_Dev_Lead | TRD-CROSS-002 | F-031 | None | M | - All user-facing strings in notification templates externalized - Hindi + English templates - Language selection based on user preference |
| S7-05 | Push notification handling (Flutter) | Frontend | Flutter_Dev_Lead | TRD-NOTIF-001 | F-021 | S7-01 | M | - FCM SDK integration - Foreground/background notification handling - Tap notification navigates to relevant screen - FCM token registration on login |
| S7-06 | Notification preferences settings UI | Frontend | Flutter_Dev_Lead | TRD-NOTIF-002 | F-021 | S7-05 | S | - Settings screen with toggle per notification category - Syncs with backend preferences API - Changes take effect immediately |
| S7-07 | Onboarding walkthrough UI | Frontend | Flutter_Dev_Lead | TRD-CROSS-003 | F-024 | S2-05 | M | - 4-step tooltip tour on first visit to "News By You" - Steps: submit, AI preview, editorial review, reputation - Skippable at any point - Final step: accept community guidelines - Does not repeat for returning users |
| S7-08 | i18n setup (Flutter ARB files) | Frontend | Flutter_Dev_Lead | TRD-CROSS-002 | F-031 | S0-01 | L | - All UI strings externalized to ARB files - Hindi + English translations - Language switcher in settings - Locale detection from device - RTL layout support scaffold |
| S7-09 | Notification service integration tests | Backend | Python_QA | TRD-NOTIF-001, 002 | F-021 | S7-03 | M | - FCM send tested (mock) for each trigger type - Delivery timing tested (< 60s) - Preference filtering tested - Batch notification logic tested |
| S7-10 | Notification and onboarding widget tests | Frontend | Flutter_QA | TRD-NOTIF-001, TRD-CROSS-003 | F-021, F-024 | S7-07 | S | - Notification tap navigation tested - Onboarding flow completion tested - Skip functionality tested - i18n string rendering tested |

**Parallel Groups:**
- Group A (Backend): S7-01 -> S7-02 -> S7-03, S7-04 (parallel to S7-01)
- Group B (Frontend): S7-05 -> S7-06 (after S7-01), S7-07 + S7-08 (independent, parallel)
- Group C (QA): S7-09, S7-10 (after dev)

**Cross-Stack Handshake:** S7-01 (Backend FCM) <-> S7-05 (Frontend FCM) -- token registration + message format alignment

---

## Sprint 8: Platform Safety and Privacy

**Goal:** Rate limiting, data privacy, abuse prevention, analytics infrastructure
**Duration:** 2 weeks

| ID | Title | Stack | Agent | TRD Ref | Feature Ref | Dependencies | Complexity | Acceptance Criteria |
|----|-------|-------|-------|---------|-------------|-------------|------------|---------------------|
| S8-01 | Rate limiting middleware | Backend | Python_Dev_Lead | TRD-PLAT-005 | F-033 | S1-02, S2-01 | M | - Per-user submission limits: Basic=5/day, Approved=10/day, Trusted=20/day - API rate limiting: 60 req/min/user, 1000 req/min/IP - Returns HTTP 429 with Retry-After header - Confirmation spam prevention (1 per user per story) |
| S8-02 | Bot detection and account flagging | Backend | Python_Dev_Lead | TRD-PLAT-005 | F-033 | S8-01 | M | - Pattern detection: rapid submissions, identical content, unusual timing - Flagged accounts enter review queue - Temporary restriction until admin review |
| S8-03 | Data privacy: account deletion API | Backend | Python_Dev_Lead | TRD-PLAT-004 | F-032 | S1-03 | M | - DELETE /api/v1/users/me schedules deletion (30-day grace period) - During grace period: reactivation possible via login - After grace period: personal data purged, stories anonymized to "Former Contributor" - KYC documents deleted |
| S8-04 | Data privacy: consent management | Backend | Python_Dev_Lead | TRD-PLAT-004 | F-032 | S1-01 | S | - Consent recorded at registration - GET /api/v1/users/me/consent returns consent status - PUT /api/v1/users/me/consent allows revocation - Revocation triggers data retention policy |
| S8-05 | Data privacy: audit trail for data access | Backend | Python_Dev_Lead | TRD-PLAT-004 | F-032 | S1-02 | M | - All internal data access logged: accessor, timestamp, data accessed, action - Immutable audit log (append-only table) - KYC document access specially logged |
| S8-06 | Analytics event tracking infrastructure | Backend | Python_Dev_Lead | TRD-PLAT-003 | F-029 | S2-01, S5-01 | L | - Event ingestion API: POST /api/v1/events - Event schema: action_type, timestamp, hashed_user_id, metadata - Supports: submission funnel, editor actions, engagement events - Events stored for dashboard consumption |
| S8-07 | Rate limit feedback UI | Frontend | Flutter_Dev_Lead | TRD-PLAT-005 | F-033 | S8-01 | S | - Display rate limit reached message when 429 received - Show remaining submissions count on form - Warning when approaching daily limit |
| S8-08 | Account deletion UI in settings | Frontend | Flutter_Dev_Lead | TRD-PLAT-004 | F-032 | S8-03 | S | - "Delete Account" button in settings - Confirmation dialog explaining 30-day grace period - Post-deletion: redirect to login screen |
| S8-09 | Analytics event SDK integration (Flutter) | Frontend | Flutter_Dev_Lead | TRD-PLAT-003 | F-029 | S8-06 | M | - Analytics events fired for: form_open, submit_tapped, feed_scroll, card_tap, like, confirm, search, session_start/end - Events batched and sent to backend - Offline events queued and sent on reconnect |
| S8-10 | Privacy and rate limiting integration tests | Backend | Python_QA | TRD-PLAT-004, 005 | F-032, F-033 | S8-05 | M | - Rate limits enforced correctly per level - Account deletion lifecycle tested - Consent management tested - Audit trail completeness verified - Bot detection patterns tested |
| S8-11 | Analytics event tests | Backend | Python_QA | TRD-PLAT-003 | F-029 | S8-06 | S | - Event ingestion tested - Funnel events tracked correctly - Event schema validation tested |

**Parallel Groups:**
- Group A (Backend): S8-01 -> S8-02 (rate limiting), S8-03 -> S8-04 -> S8-05 (privacy, parallel to A), S8-06 (analytics, parallel)
- Group B (Frontend): S8-07 (after S8-01), S8-08 (after S8-03), S8-09 (after S8-06) -- all parallel
- Group C (QA): S8-10, S8-11 (after dev)

---

## Sprint 9: Administration and Advanced Features

**Goal:** Admin panel, editor analytics, trending, accessibility audit, final hardening
**Duration:** 2 weeks

| ID | Title | Stack | Agent | TRD Ref | Feature Ref | Dependencies | Complexity | Acceptance Criteria |
|----|-------|-------|-------|---------|-------------|-------------|------------|---------------------|
| S9-01 | Admin moderation panel API | Backend | Python_Dev_Lead | TRD-PLAT-001 | F-037 | S1-05, S4-04, S6-05 | L | - GET /api/v1/admin/moderation/reports (reported content queue) - GET /api/v1/admin/moderation/kyc (KYC review queue) - GET /api/v1/admin/moderation/appeals (appeal escalations) - POST /api/v1/admin/creators/{id}/action (warn/suspend/ban) - PUT /api/v1/admin/config (system parameters) - All actions audit-logged |
| S9-02 | Editor dashboard analytics API | Backend | Python_Dev_Lead | TRD-PLAT-002 | F-034 | S8-06 | M | - GET /api/v1/editor/dashboard - Returns: pending_count, avg_approval_time, rejection_rate, published_today, top_cities - City filter support - AI accuracy metrics: safety false positive rate, duplicate false positive rate |
| S9-03 | Trending stories algorithm | Backend | Python_Dev_Lead | TRD-FEED-005 | F-020 | S6-01, S6-02 | M | - Trending score: (confirmations * 3) + (likes * 1) + recency_decay - Computed per city when filtered - Stories above threshold get trending flag - No trending section if no qualifying stories |
| S9-04 | Admin moderation panel UI | Frontend | Flutter_Dev_Lead | TRD-PLAT-001 | F-037 | S9-01 | L | - Tab-based queues: reports, KYC, appeals, flagged accounts - Action buttons: warn, suspend (7d/30d), ban - System config editor for rate limits and AI thresholds - All actions with confirmation dialogs |
| S9-05 | Editor analytics dashboard UI | Frontend | Flutter_Dev_Lead | TRD-PLAT-002 | F-034 | S9-02 | M | - Dashboard with key metrics cards - City filter - AI accuracy section - Auto-refresh every 5 min |
| S9-06 | Trending badge and feed integration | Frontend | Flutter_Dev_Lead | TRD-FEED-005 | F-020 | S9-03 | S | - "Trending" badge on qualifying story cards - Trending stories promoted in feed position - Trending section hidden when no qualifying stories |
| S9-07 | Accessibility audit and fixes | Frontend | Flutter_Dev_Lead | TRD-CROSS-001 | F-030 | S5-08, S6-07 | L | - Full VoiceOver/TalkBack audit of all screens - Fix all contrast ratio violations - Add missing Semantics labels - Ensure keyboard/switch control navigation - Add alt text to all story images |
| S9-08 | Admin panel integration tests | Backend | Python_QA | TRD-PLAT-001 | F-037 | S9-01 | M | - All moderation queues tested - Creator warn/suspend/ban tested - Config changes tested - Audit logging verified |
| S9-09 | Trending algorithm tests | Backend | Python_QA | TRD-FEED-005 | F-020 | S9-03 | S | - Scoring formula verified - City-scoped trending tested - Threshold behavior tested |
| S9-10 | End-to-end integration testing (full flow) | Both | Both QA | All TRDs | All features | S9-07 | L | - Full flow: register -> submit -> AI process -> editor review -> publish -> feed -> confirm/like -> share - Cross-stack data consistency verified - Performance under load tested - Accessibility pass verified |
| S9-11 | Accessibility widget tests | Frontend | Flutter_QA | TRD-CROSS-001 | F-030 | S9-07 | M | - Semantics labels on all interactive elements - Contrast ratios verified programmatically - Focus order logical on every screen |

**Parallel Groups:**
- Group A (Backend): S9-01 + S9-02 + S9-03 (all parallel)
- Group B (Frontend): S9-04 (after S9-01), S9-05 (after S9-02), S9-06 (after S9-03), S9-07 (independent)
- Group C (QA): S9-08, S9-09, S9-10, S9-11 (after dev)

---

## Summary Statistics

| Sprint | Tasks | Backend | Frontend | QA | Key Features |
|--------|-------|---------|----------|-----|-------------|
| S0 | 8 | 5 | 3 | 0 | Foundation, scaffolding |
| S1 | 12 | 5 | 4 | 3 | Auth, creator levels, KYC |
| S2 | 10 | 4 | 4 | 2 | Submission, images, dashboard, drafts |
| S3 | 12 | 9 | 2 | 1 | AI rephrase, safety, tags, duplicates, confidence |
| S4 | 10 | 4 | 4 | 2 | Editor queue, actions, false news, appeals |
| S5 | 13 | 5 | 6 | 2 | Feed, detail, search, location, sharing |
| S6 | 11 | 5 | 4 | 2 | Confirm, like, reputation, profiles, blocking |
| S7 | 10 | 4 | 4 | 2 | Notifications, onboarding, i18n |
| S8 | 11 | 6 | 3 | 2 | Rate limiting, privacy, analytics |
| S9 | 11 | 3 | 4 | 4 | Admin, editor analytics, trending, accessibility |
| **Total** | **108** | **50** | **38** | **20** | **All 38 features covered** |

---

## Feature-to-Sprint Mapping (all 38 features)

| Feature | Sprint(s) | Task IDs |
|---------|-----------|----------|
| F-001 | S1 | S1-01, S1-02, S1-06, S1-07, S1-10, S1-11 |
| F-002 | S1, S6 | S1-03, S1-04, S1-08, S6-09 |
| F-003 | S2 | S2-01, S2-03, S2-05, S2-09, S2-10 |
| F-004 | S2 | S2-04, S2-06 |
| F-005 | S3 | S3-01, S3-02, S3-10, S3-12 |
| F-006 | S3 | S3-04, S3-05, S3-12 |
| F-007 | S3 | S3-06, S3-11, S3-12 |
| F-008 | S3 | S3-07, S3-12 |
| F-009 | S3 | S3-08, S3-09, S3-12 |
| F-010 | S4 | S4-01, S4-05, S4-09, S4-10 |
| F-011 | S4 | S4-02, S4-06, S4-09, S4-10 |
| F-012 | S4 | S4-03, S4-07, S4-09 |
| F-013 | S5 | S5-01, S5-06, S5-07, S5-12, S5-13 |
| F-014 | S5 | S5-02, S5-08, S5-13 |
| F-015 | S6 | S6-01, S6-06, S6-10, S6-11 |
| F-016 | S6 | S6-02, S6-06, S6-10, S6-11 |
| F-017 | S6 | S6-03, S6-09, S6-10 |
| F-018 | S6 | S6-04, S6-07, S6-10 |
| F-019 | S2 | S2-07, S2-08 |
| F-020 | S9 | S9-03, S9-06, S9-09 |
| F-021 | S7 | S7-01, S7-02, S7-03, S7-05, S7-06, S7-09, S7-10 |
| F-022 | S1 | S1-05, S1-09, S1-12 |
| F-023 | S4 | S4-04, S4-08, S4-09 |
| F-024 | S7 | S7-07, S7-10 |
| F-025 | S5 | S5-03, S5-09 |
| F-026 | S6 | S6-05, S6-08, S6-10 |
| F-027 | S2 | S2-02, S2-09 |
| F-028 | S2 | S2-08 |
| F-029 | S8 | S8-06, S8-09, S8-11 |
| F-030 | S0, S9 | S0-08, S9-07, S9-11 |
| F-031 | S7 | S7-04, S7-08, S7-10 |
| F-032 | S8 | S8-03, S8-04, S8-05, S8-08, S8-10 |
| F-033 | S8 | S8-01, S8-02, S8-07, S8-10 |
| F-034 | S9 | S9-02, S9-05 |
| F-035 | S5 | S5-05, S5-11 |
| F-036 | S5 | S5-04, S5-10 |
| F-037 | S9 | S9-01, S9-04, S9-08 |
| F-038 | S3 | S3-01, S3-03, S3-12 |

---

## Cross-Stack Handshake Points (Master_Orchestrator coordination required)

| Sprint | Backend Task | Frontend Task | Handshake Description |
|--------|-------------|---------------|----------------------|
| S1 | S1-01 (Login API) | S1-06 (Login UI) | Auth endpoint contract alignment |
| S2 | S2-01 (Submission API) | S2-05 (Submission Form) | Submission endpoint contract alignment |
| S3 | S3-09 (AI Pipeline) | S3-10 (AI Preview) | Pipeline status polling / webhook format |
| S4 | S4-01/S4-02 (Editor APIs) | S4-05/S4-06 (Editor UI) | Queue + action endpoint alignment |
| S5 | S5-01/S5-02 (Feed APIs) | S5-06/S5-07/S5-08 (Feed UI) | Feed + detail response format |
| S6 | S6-01/S6-02 (Confirm/Like) | S6-06 (Confirm/Like UI) | Optimistic update sync pattern |
| S7 | S7-01 (FCM Service) | S7-05 (FCM Handler) | Token registration + message format |

---

**END OF SPRINT TASK LIST v2.0**
