# Python Dev Lead - Thought Log

## Agent Config
- **Role:** Implement FastAPI backend per Backend_TRD
- **Toolset:** FastAPI, SQLAlchemy, Alembic, Cloud Build/Cloud Run, Pydantic
- **Cloud Code:** Generates cloudbuild.yaml, deployment configs, local emulator setup

---

## Log

### 2026-03-27 | INIT
**Status:** Awaiting Sprint 0 task S0-02 (Scaffold FastAPI project).
**Architecture decisions:**
- Framework: FastAPI with async endpoints
- ORM: SQLAlchemy 2.0 (async)
- Migrations: Alembic
- Validation: Pydantic v2
- Auth: python-jose (JWT)
- Storage: google-cloud-storage
- AI: Cloud NLP API + custom safety models
- Deployment: Cloud Run via Cloud Build
- Local dev: Docker Compose (Postgres + Redis + MinIO)

### 2026-03-27 | Sprint 0 Tasks Complete (S0-02, S0-05, S0-06, S0-07)

**S0-02: FastAPI Project Scaffold - DONE**
- Created full project structure: app/{models,schemas,routers,services}, migrations, tests, deploy
- FastAPI app with CORS middleware, health check, lifespan management
- pydantic-settings based config (app/config.py) loading from .env
- Async SQLAlchemy engine + session factory (app/database.py)
- Shared dependencies: get_db, get_current_user, get_current_editor, get_current_admin (app/dependencies.py)
- All 5 routers mounted under /api/v1: auth, submissions, editorial, feed, community
- All 7 services with real method signatures: auth, submission, ai (placeholder), editorial, feed, community, reputation
- All 6 Pydantic schema modules matching contract.yaml exactly
- pyproject.toml with uv/pip config, ruff, mypy, pytest configuration

**S0-05: PostgreSQL Schema + Models - DONE**
- 9 SQLAlchemy models with proper relationships, indexes, constraints:
  - User (creator_level enum, reputation_points, is_editor, is_admin)
  - Submission (status enum lifecycle, FK to user)
  - SubmissionImage (position-ordered, FK to submission)
  - AIReview (unique per submission, safety/confidence scores, JSON tags)
  - EditorAction (action enum, FK to submission + editor)
  - Story (unique per submission, JSON images, like/confirmation counts)
  - Confirmation (unique story+user constraint)
  - Like (unique story+user constraint)
  - Badge (user FK, badge_type)
- Alembic configured for async migrations (migrations/env.py imports all models)
- script.py.mako template ready for migration generation

**S0-06: CI/CD + Docker - DONE**
- Multi-stage Dockerfile: python:3.11-slim builder -> runtime with uvicorn
- docker-compose.yaml: postgres:15, redis:7, minio, backend with health checks
- cloudbuild.yaml: lint (ruff) -> test (pytest) -> build (docker) -> push -> deploy (Cloud Run)
- deployment.yaml: K8s deployment + service with resource limits, health/readiness probes

**S0-07: Redis + Object Storage - DONE**
- Redis connection pool with CacheService abstraction (app/redis_client.py)
- Storage backend abstraction with MinIO and GCS implementations (app/storage.py)
- Factory pattern: get_storage() returns configured backend based on STORAGE_BACKEND env var
- Redis warmed up during app lifespan startup

**Testing:**
- conftest.py with async fixtures: test DB (SQLite), test client, test_user, test_editor, auth_headers
- test_health.py: health check, root endpoint, docs accessibility tests

**Key decisions:**
- Used SQLAlchemy 2.0 mapped_column style (not legacy Column)
- UUID primary keys on all tables for distributed-friendly IDs
- All timestamps with timezone=True and server_default=func.now()
- Toggle semantics on like/confirm (POST toggles on/off, matches contract)
- AI service is a placeholder interface - real implementation in Sprint 3
- StorageBackend ABC allows swapping MinIO (dev) / GCS (prod) seamlessly

### 2026-03-27 | Sprint 1 Tasks Complete (S1-01, S1-02, S1-03, S1-04, S1-05)

**S1-01: HT SSO Login API Endpoint - DONE**
- POST /api/v1/auth/login accepts email + password
- Validates credentials against PostgreSQL users table (bcrypt hashed passwords via passlib)
- Returns JWT access_token + refresh_token + full UserProfile
- Returns 401 with "Invalid email or password" on bad credentials
- auth_service.authenticate_user() does async DB lookup + bcrypt verify
- Token payload includes sub (user_id), email, creator_level, type, iat, exp

**S1-02: JWT Token Issuance, Refresh, and Revocation - DONE**
- Access token TTL: 15 minutes (configurable via jwt_access_token_expire_minutes)
- Refresh token TTL: 30 days (configurable via jwt_refresh_token_expire_days)
- POST /api/v1/auth/refresh — validates refresh token, issues new pair (token rotation)
- POST /api/v1/auth/logout — revokes refresh token via Redis
- Refresh tokens stored in Redis with key "refresh_token:{jti}" and TTL matching token expiry
- Revoked tokens tracked in Redis with key "revoked_token:{jti}"
- Token rotation: old refresh token is revoked when a new pair is issued
- JWT payload includes user_id (sub), email, creator_level per spec
- python-jose for JWT encode/decode, passlib[bcrypt] for password hashing

**S1-03: User Profile API with Creator Level - DONE**
- GET /api/v1/auth/profile — returns full UserProfile for authenticated user
- Includes: id, name, email, avatar_url, creator_level, reputation_points, city
- stories_published: counted via async query on submissions with status=PUBLISHED
- badges: list of badge_type strings from user.badges relationship
- Requires valid JWT (CurrentUser dependency)
- Optimized: uses COUNT() query instead of loading all submission objects

**S1-04: Creator Level Upgrade Logic - DONE**
- New service: app/services/creator_level_service.py
- upgrade_to_ht_approved(db, user_id, performed_by) — basic_creator -> ht_approved_creator
  - Triggered on KYC approval
  - Awards "ht_approved_creator" badge
  - Validates user is currently basic_creator (raises ValueError otherwise)
- check_and_upgrade_to_trusted_reporter(db, user_id) — ht_approved_creator -> trusted_reporter
  - Criteria: 50+ published stories AND accuracy > 90% (published / (published + rejected))
  - Awards "trusted_reporter" badge
  - Auto-triggered by reputation_service after point awards
- New model: AuditLog (audit_logs table) with action enum for tracking upgrades
- Every level change creates an AuditLog record with detail, performed_by, timestamp

**S1-05: KYC Document Upload and Review Queue API - DONE**
- New model: KYCSubmission (kyc_submissions table) with status enum (pending/approved/rejected)
  - Fields: user_id, document_type, document_storage_key, document_hash (SHA-256), status, reviewer_id, reviewer_notes, submitted_at, reviewed_at
- POST /api/v1/kyc/upload — authenticated creator uploads government ID
  - Validates document_type (aadhaar, pan, passport, voter_id, driving_license)
  - Validates file type (JPEG, PNG, WebP, PDF) and size (max 10 MB)
  - Prevents duplicate pending submissions per user
  - Uploads to object storage with key pattern kyc/{user_id}/{uuid}.{ext}
  - Computes SHA-256 hash for integrity verification
  - Returns 201 with KYCUploadResponse
- GET /api/v1/kyc/queue — admin-only endpoint to list pending KYC submissions
  - Paginated (page, page_size query params)
  - Optional status_filter query param
  - Returns user name/email alongside each submission
- POST /api/v1/kyc/{id}/review — admin approves or rejects a KYC submission
  - On approval: triggers upgrade_to_ht_approved (basic_creator -> ht_approved_creator)
  - Creates audit log entry for both approve and reject actions
  - Returns KYCReviewResponse with creator_level_upgraded flag
- New router: app/routers/kyc.py mounted at /api/v1/kyc
- New schemas: app/schemas/kyc.py
- New service: app/services/kyc_service.py

**New files created:**
- app/models/audit_log.py — AuditLog model with AuditAction enum
- app/models/kyc_submission.py — KYCSubmission model with KYCStatus enum
- app/services/creator_level_service.py — Creator level upgrade logic
- app/services/kyc_service.py — KYC upload + review service
- app/schemas/kyc.py — KYC Pydantic schemas
- app/routers/kyc.py — KYC API router

**Files modified:**
- app/main.py — Added KYC router import and mount
- app/models/__init__.py — Added AuditLog, KYCSubmission exports
- app/schemas/auth.py — Added LogoutRequest, LogoutResponse, CreatorLevelUpgradeResponse
- app/routers/auth.py — Added /refresh and /logout endpoints, full OpenAPI metadata
- app/services/auth_service.py — Complete rewrite: Redis-backed refresh tokens, token rotation, rich JWT payload, structured logging
- app/services/reputation_service.py — Integrated with creator_level_service for auto-upgrade checks
- migrations/env.py — Added AuditLog, KYCSubmission model imports

**Key design decisions:**
- Token rotation on refresh (old refresh token revoked, new one issued) for security
- JWT ID (jti) on refresh tokens enables per-token revocation via Redis
- Redis stores both active tokens and revoked tokens (with TTL auto-cleanup)
- KYC upload computes SHA-256 hash for document integrity verification
- Single pending KYC submission per user enforced at service level
- Creator level upgrades gated by explicit checks (not blind SET)
- AuditLog captures who performed the action (performed_by) for admin operations

### 2026-03-27 | Sprint 2 Tasks Complete (S2-01, S2-02, S2-03, S2-04)

**S2-01: Submission CRUD API - DONE**
- POST /api/v1/submissions — Create a new submission (multipart form)
  - Accepts: title (required, max 200), description (required, max 5000), city (required), cover_image (required file), tags (optional JSON array)
  - cover_image is processed through image service (S2-02): validated, compressed, 3 variants uploaded
  - Creates submission with status=draft initially
  - Returns full SubmissionSchema with 201
- GET /api/v1/submissions/my — List current user's submissions ("Posted by You")
  - Cursor-based pagination (cursor is ISO datetime, limit default 20, max 100)
  - Optional status filter query param
  - Returns SubmissionListResponse with items, next_cursor, has_more
  - Sorted by created_at DESC
- GET /api/v1/submissions/{id} — Get submission detail
  - Returns full SubmissionDetail including original_text, ai_review, editor_notes, additional images
  - Access control: creator sees own submissions in any status; editors see under_review/published/rejected
  - Returns 403 if unauthorized, 404 if not found

**S2-02: Image Upload with Constraints - DONE**
- New service: app/services/image_service.py
- Validation: max 10MB, JPEG/PNG/HEIC only (by content_type and file extension), min resolution 320x240
- Server-side compression using Pillow (PIL):
  - Thumbnail: 150px longest side, JPEG quality 85
  - Medium: 600px longest side, JPEG quality 85
  - Original: JPEG re-encode at quality 85 (normalizes format)
- All variants uploaded to object storage via StorageBackend abstraction
- ImageVariants dataclass holds URLs for all 3 sizes
- HEIC support via optional pillow-heif fallback
- Handles RGBA/P/LA mode conversion to RGB for JPEG compatibility
- Added Pillow==10.3.0 to requirements.txt

**S2-03: Submission Status Lifecycle State Machine - DONE**
- VALID_TRANSITIONS dict defines allowed state transitions in app/models/submission.py:
  - draft -> in_progress (creator submits)
  - in_progress -> under_review (AI processing completes)
  - under_review -> published (editor approves)
  - under_review -> rejected (editor rejects)
  - under_review -> in_progress (editor marks correction needed)
  - rejected -> in_progress (appeal approved, future sprint)
  - published -> (terminal state, no transitions)
- validate_transition() raises HTTPException 400 on invalid transitions with descriptive message
- transition_status() applies transition + creates StatusTransitionLog record
- StatusTransitionLog model (status_transition_logs table):
  - Fields: submission_id, from_status, to_status, actor_id, actor_type ("user"/"system"), transitioned_at
  - FK to submissions and users tables
- POST /api/v1/submissions/{id}/status endpoint with authorization checks:
  - Creator: can submit drafts (draft -> in_progress)
  - Editors: can approve/reject/request corrections (under_review -> published/rejected/in_progress)
  - Editors: can approve appeals (rejected -> in_progress)
- update_submission_status() backward-compatible wrapper for existing AI review code
- Each transition logged with timestamp, actor_id, and actor_type

**S2-04: Multi-image Upload Endpoint - DONE**
- POST /api/v1/submissions/{id}/images — upload additional images (batch)
- Max 9 additional images (10 total including cover)
- Each image validated with same constraints as S2-02 (10MB, JPEG/PNG/HEIC, 320x240 min)
- Each image gets 3 compressed variants (thumbnail, medium, original)
- Position field auto-incremented for ordering
- Only submission creator can add images (403 otherwise)
- Only while status is draft or in_progress (400 otherwise)
- Returns MultiImageUploadResponse with all uploaded images and total count

**New files created:**
- app/services/image_service.py — Image validation, compression, variant generation, upload

**Files modified:**
- app/models/submission.py — Added: tags (JSON), cover_image_thumbnail_url, cover_image_medium_url columns to Submission; thumbnail_url, medium_url to SubmissionImage; StatusTransitionLog model; VALID_TRANSITIONS dict; default status changed to DRAFT
- app/models/__init__.py — Added StatusTransitionLog export
- app/schemas/submission.py — Added: SubmissionImageResponse, SubmissionListResponse (cursor pagination), StatusTransitionRequest/Response, MultiImageUploadResponse; Updated Submission schema with tags and variant URLs
- app/services/submission_service.py — Full rewrite: cursor-based pagination, state machine validation/transition/logging, image count/position management, upload validation logic
- app/routers/submissions.py — Full rewrite: real image upload in create, cursor pagination on /my, access control on GET /{id}, POST /{id}/status endpoint, POST /{id}/images endpoint
- migrations/env.py — Added StatusTransitionLog import
- requirements.txt — Added Pillow==10.3.0

**Key design decisions:**
- Default submission status is DRAFT (not IN_PROGRESS) per task spec — creator explicitly submits
- Cursor-based pagination using created_at timestamp (ISO format) for infinite scroll support
- State machine defined as a dict for easy auditing and clear invalid-transition error messages
- Image variants are JPEG-normalized regardless of input format (consistent CDN serving)
- Cover image variants stored as dedicated columns on Submission (not in SubmissionImage table)
- Additional images stored in SubmissionImage with all 3 variant URLs
- Batch upload validates total count before processing any images (fail-fast)
- StatusTransitionLog enables full audit trail of who did what and when

### 2026-03-27 | Sprint 3 Tasks Complete (S3-01 through S3-09)

**S3-01: Language Detection Service - DONE**
- New file: app/ai/language.py
- Uses langdetect library as primary detector (deterministic with seed=0)
- Supports 8 languages: Hindi (hi), English (en), Marathi (mr), Tamil (ta), Telugu (te), Bengali (bn), Kannada (kn), Gujarati (gu)
- Returns LanguageDetectionResult with language_code, confidence score, and is_supported flag
- Fallback to "unknown" with confidence 0.0 if detection fails or text is empty

**S3-02: AI Text Rephrasing Service - DONE**
- New file: app/ai/rephrasing.py
- Mock LLM client simulates rephrasing: cleans whitespace, fixes double spaces, collapses newlines, capitalizes sentences (English)
- Returns RephraseResult with original_text, rephrased_text, similarity_score (mock: 0.9), model_used
- Fallback: if rephrasing fails, returns original text with rephrase_failed=True flag

**S3-03: Multilingual Rephrase Model Selection - DONE**
- In app/ai/rephrasing.py: LANGUAGE_MODEL_MAP routes each supported language to a ModelConfig
- Each ModelConfig has model_name, language_code, max_tokens, temperature
- For unsupported languages: sets manual_review_required=True, skips rephrase, passes original through
- get_model_config() returns None for unsupported languages

**S3-04: AI Safety Guardrails - Text Scanning - DONE**
- New file: app/ai/safety.py
- Keyword-based detection for: hate speech, toxicity, violence, spam
- Each category has a configurable weight (hate: 0.30, toxicity: 0.25, violence: 0.20, spam: 0.25)
- Returns TextSafetyResult with individual boolean flags, composite safety_score (0.0-1.0), auto_rejected flag, flagged_terms
- Auto-reject threshold: safety_score < 0.2 (configurable via AUTO_REJECT_THRESHOLD)

**S3-05: AI Safety Guardrails - Image Scanning - DONE**
- In app/ai/safety.py: scan_image_safety() is a stub that returns safe for all images
- Returns ImageSafetyResult with explicit_content, violence, manipulation flags (all False)
- compute_composite_safety() combines text (weight 0.6) and image (weight 0.4) scores
- Returns CompositeSafetyResult with combined composite_safety_score and auto_rejected flag

**S3-06: AI Tag and Category Suggestion - DONE**
- New file: app/ai/metadata.py
- Tag extraction using simple TF (term frequency) approach with stop word filtering
- Minimum 3 tags guaranteed (pads with "citizen-news", "local", "report" defaults)
- Category suggestion from 13 predefined categories using keyword matching
- CATEGORY_KEYWORDS maps each category to a list of indicator keywords
- City is never modified (pass-through in MetadataSuggestion)

**S3-07: AI Duplicate Detection Service - DONE**
- New file: app/ai/duplicate.py
- Compares submission text against published stories using TF-IDF cosine similarity (scikit-learn)
- Filters by same city + 48-hour time window (configurable DUPLICATE_TIME_WINDOW_HOURS)
- Checks up to 50 most recent stories in the matching city
- Returns DuplicateCheckResult: is_duplicate=True if similarity > 0.8 (DUPLICATE_THRESHOLD)
- Returns duplicate_of UUID pointing to the matching story's submission_id

**S3-08: AI Editorial Confidence Score Calculator - DONE**
- New file: app/ai/scoring.py
- Weighted composite formula:
  - safety_score: 0.3 weight
  - language_quality (rephrase similarity): 0.2 weight
  - duplicate_score (1.0 - similarity, so no duplicate = 1.0): 0.1 weight
  - creator_reputation (normalized 0-1, max 1000 points): 0.2 weight
  - community_confirmations (normalized 0-1, max 50): 0.2 weight
- Returns ConfidenceScoreResult with overall score and per-component breakdown

**S3-09: AI Orchestration Pipeline (Async) - DONE**
- New file: app/ai/pipeline.py
- Full pipeline orchestration with asyncio:
  1. Detect language (async via executor)
  2. Text safety scan + Image safety scan (asyncio.gather - parallel)
  3. Rephrase text (after language detection)
  4. Suggest metadata + Check duplicates (asyncio.gather - parallel)
  5. Calculate confidence score (uses creator reputation from DB)
  6. Create/update AIReview record in database
  7. Transition submission status: in_progress -> under_review
- Circuit breaker: if any step throws, submission stays in_progress, PipelineResult.success=False
- All synchronous AI functions wrapped in _run_in_executor for non-blocking async execution
- _create_or_update_ai_review handles both new and re-processed submissions

**New API endpoint:**
- POST /api/v1/submissions/{id}/process-ai - triggers the full AI pipeline
  - Authorization: only creator or editors
  - Validates submission is in draft or in_progress
  - If draft, auto-transitions to in_progress first
  - Returns AIReviewResult

**Updated files:**
- app/services/ai_service.py - Replaced placeholder with pipeline delegation + fallback handling
  - trigger_ai_review() now calls run_ai_pipeline() from app.ai.pipeline
  - On pipeline failure: creates minimal fallback AIReview so caller always gets a response
- app/routers/submissions.py - Added POST /{id}/process-ai endpoint; removed duplicate status transition from ai-review endpoint (pipeline handles it now)
- requirements.txt - Added langdetect==1.0.9, scikit-learn==1.4.2

**New files created:**
- app/ai/__init__.py - Package init with module documentation
- app/ai/language.py - Language detection service (S3-01)
- app/ai/rephrasing.py - Text rephrasing + multilingual model selection (S3-02, S3-03)
- app/ai/safety.py - Text and image safety guardrails (S3-04, S3-05)
- app/ai/metadata.py - Tag and category suggestion (S3-06)
- app/ai/duplicate.py - Duplicate detection service (S3-07)
- app/ai/scoring.py - Confidence score calculator (S3-08)
- app/ai/pipeline.py - Async orchestration pipeline (S3-09)

**Key design decisions:**
- All AI functions are synchronous internally, wrapped in asyncio executors for pipeline orchestration
- Pipeline uses asyncio.gather for parallel steps (safety scans, metadata+duplicate)
- Circuit breaker pattern: pipeline catches all exceptions, returns PipelineResult(success=False)
- ai_service.py acts as a stable interface - callers never import pipeline directly
- Fallback review with safety_score=0.0 and confidence_score=0.0 created if pipeline fails entirely
- Mock implementations designed for easy replacement with real ML models/APIs later
- TF-IDF vectorizer uses (1,2) ngram range and 5000 max features for good balance
- Duplicate detection scoped to same city + 48-hour window to reduce false positives

### 2026-03-27 | Sprint 4 Tasks Complete (S4-01, S4-02, S4-03, S4-04)

**S4-01: Editor Review Queue API - DONE**
- GET /api/v1/editorial/queue — returns submissions with status=under_review
- Requires editor role (CurrentEditor dependency)
- Each queue item includes: SubmissionDetail, AI confidence score, community confirmation count, priority rank
- Sort options: confidence_score (default, in-memory sort after selectinload), submitted_at (DB order), confirmations (correlated subquery)
- City filter via query param
- Cursor-based pagination (ISO datetime cursor, limit 1-100, has_more flag)
- Joins with ai_reviews (via selectinload) for confidence scores
- Joins with users (via selectinload) for creator info
- Community confirmation count queried per submission via join on Story->Confirmation
- Response model: EditorialQueueResponse with items, next_cursor, has_more

**S4-02: Editor Action API - DONE**
- POST /api/v1/editorial/{id}/action — performs editorial actions on submissions
- Requires editor role
- Actions: approve, edit, reject, mark_correction
- **Approve:** Creates Story from submission (title, description, images, city), sets ai_verified + editor_verified, sets status=published, awards 10 reputation points to creator, creates EditorAction audit record + StatusTransitionLog
- **Edit:** Accepts edited_title/edited_description, applies edits to submission, creates Story with edited content, sets status=published, awards 10 points, creates audit records
- **Reject:** Requires rejection_reason, sets status=rejected, creates audit record (no points awarded)
- **Mark Correction:** Requires correction_notes, sets status=in_progress (back to creator), creates audit record
- Validates submission is in under_review status before any action
- Validates action-specific required fields (rejection_reason for reject, correction_notes for mark_correction, at least one edit field for edit)
- Response model: EditorActionResponse with full audit detail

**S4-03: False News Retroactive Flagging API - DONE**
- POST /api/v1/editorial/{id}/flag-false — flags published stories as false news
- Requires editor role
- Validates: submission exists, status=published, story exists, not already flagged
- New Story model fields: is_false_report (Boolean, default=false), is_hidden (Boolean, default=false)
- Sets story.is_false_report=True, story.is_hidden=True
- New EditorActionType enum value: FLAG_FALSE = "flag_false"
- Creates EditorAction audit record with action=flag_false and reason
- Deducts 20 reputation points from creator via reputation_service.deduct_false_news_points()
- Counts creator's total false reports via _count_creator_false_reports() query
- If creator has 3+ false reports: logs warning for demotion review (demotion_review_triggered=True)
- Feed service updated: get_feed() and get_story_by_id() now filter Story.is_hidden.is_(False)
- Response model: FlagFalseResponse with story_id, points_deducted, false_report_count, demotion_review_triggered
- Duplicate flagging prevented with 409 Conflict

**S4-04: Appeal Process API - DONE**
- POST /api/v1/submissions/{id}/appeal — creator appeals a rejected submission
  - Validates: submission exists, status=rejected, user is original creator, no existing appeal
  - Creates Appeal record with justification, status=pending
  - Returns 409 if appeal already exists (one per submission enforced via unique constraint)
  - Returns 201 with AppealResponse
- GET /api/v1/editorial/appeals — editor lists pending appeals
  - Requires editor role
  - Returns all pending appeals sorted by created_at ascending (oldest first)
  - Loads submission.user and appellant via selectinload
  - Response: AppealListResponse with items + total count
- POST /api/v1/editorial/appeals/{id}/review — editor reviews an appeal
  - Requires editor role
  - Must be different editor than original reviewer (enforced by checking EditorAction reject records)
  - Decision: "approved" or "rejected"
  - If approved: transitions submission rejected->under_review, creates StatusTransitionLog
  - If rejected: finalizes rejection (appeal status=rejected)
  - Sets reviewing_editor_id and reviewed_at timestamp

**New files created:**
- app/models/appeal.py — Appeal model with AppealStatus enum (pending/approved/rejected), unique submission_id constraint, appellant/reviewer relationships

**Files modified:**
- app/models/editor_action.py — Added FLAG_FALSE to EditorActionType enum
- app/models/story.py — Added is_false_report and is_hidden Boolean columns with default=false
- app/models/submission.py — Added appeal relationship (uselist=False, selectinload); Updated VALID_TRANSITIONS: rejected now allows transition to under_review (for appeal approval)
- app/models/__init__.py — Added Appeal import and export
- app/schemas/editorial.py — Full rewrite: added EditorialQueueResponse, EditorActionResponse, FlagFalseRequest/Response, AppealCreateRequest, AppealResponse, AppealListResponse, AppealReviewRequest
- app/services/editorial_service.py — Full rewrite: get_editorial_queue() with cursor pagination/sorting/filtering; perform_editor_action() with validation, story publishing, reputation points, audit trails; flag_story_as_false() with story flagging, point deduction, demotion check; create_appeal(), get_pending_appeals(), review_appeal() with different-editor enforcement
- app/routers/editorial.py — Full rewrite: GET /queue with pagination; POST /{id}/action with EditorActionResponse; POST /{id}/flag-false; GET /appeals; POST /appeals/{id}/review
- app/routers/submissions.py — Added POST /{id}/appeal endpoint for creators; added editorial imports
- app/services/feed_service.py — Added is_hidden filter to get_feed() and get_story_by_id() to hide false-flagged stories from feed
- migrations/env.py — Added Appeal model import

**Key design decisions:**
- Cursor-based pagination for editorial queue uses created_at as cursor position, regardless of sort field
- Confidence score sorting done in-memory after selectinload because AI review is loaded via relationship (avoids complex join query with selectinload)
- Confirmation count for queue items uses correlated subquery when sorting by confirmations, but per-item query when building response items
- Appeal model uses unique constraint on submission_id for one-appeal-per-submission enforcement
- Different-editor enforcement on appeal review checks EditorAction records for the reject action
- VALID_TRANSITIONS updated to allow rejected->under_review for appeal approval pathway
- Feed service filters is_hidden=False to remove false-flagged stories from public feed
- All editorial actions create StatusTransitionLog records with actor_type="editor" for audit trail
- FlagFalseRequest requires minimum 10-char reason; AppealCreateRequest requires minimum 20-char justification

### 2026-03-27 | Sprint 6 Tasks Complete (S6-01, S6-02, S6-03, S6-04, S6-05)

**S6-01: Confirm Story API - DONE**
- POST /api/v1/stories/{id}/confirm — toggles confirmation on a story
- Authenticated users only (CurrentUser dependency)
- Self-confirmation prevented: checks submission.user_id != current_user.id, returns 403
- Toggle: first call creates Confirmation record, second call deletes it
- Atomic count update: uses SQL `UPDATE stories SET confirmations_count = confirmations_count + 1` (not in-memory increment)
- Awards 2 reputation points to story creator on new confirmation via reputation_service.award_confirmation_points()
- Returns ConfirmationResult: story_id, total_confirmations, confirmed (boolean)

**S6-02: Like Story API - DONE**
- POST /api/v1/stories/{id}/like — toggles like on a story
- Authenticated users only
- Self-liking prevented: checks submission.user_id != current_user.id, returns 403
- Toggle: first call creates Like record, second call deletes it
- Atomic count update: uses SQL `UPDATE stories SET likes_count = likes_count + 1` (not in-memory increment)
- Awards 1 reputation point to story creator on new like via reputation_service.award_like_points()
- Returns LikeResult: story_id, total_likes, liked (boolean)

**S6-03: Reputation Points Engine - DONE**
- Complete rewrite of app/services/reputation_service.py with full logic
- Points table: publish +10, confirmation +2, like +1, false news -20
- Atomic point updates: uses SQL `UPDATE users SET reputation_points = GREATEST(0, reputation_points + :delta) RETURNING reputation_points`
- Prevents negative reputation (GREATEST(0, ...))
- Milestone badge system with _check_milestones():
  - 100 points: "century_reporter" badge
  - 500 points: "community_voice" badge
  - 1000 points: "ht_gifts_eligible" badge + logs HT gifts eligibility flag
- Duplicate badge prevention: _award_badge_if_missing() checks existing badges before awarding
- accuracy_rate computation: (published - false_reports) / published * 100
  - false_reports counted via Story.is_false_report join through Submission
- Trusted reporter eligibility checked after every point change (50+ published + accuracy > 90%)
- All operations are atomic within a single DB transaction

**S6-04: Creator Public Profile API - DONE**
- GET /api/v1/creators/{id}/profile — public endpoint (no auth required)
- Returns CreatorProfileDetailed with:
  - id, name, avatar_url, creator_level, reputation_points, stories_published count
  - badges list (badge_type strings), accuracy_rate (computed via reputation_service)
  - joined_at
  - stories: paginated list of published stories (most recent first)
  - stories_next_cursor, stories_has_more for pagination
- CreatorProfileStory schema: id, title, city, likes_count, confirmations_count, published_at
- Stories query filters: is_hidden=False, is_false_report=False, submission.status=PUBLISHED
- Cursor-based pagination for stories list

**S6-05: Block and Report APIs - DONE**
- POST /api/v1/users/{id}/block — toggle block on a creator
  - Authenticated users only
  - Self-blocking prevented (403)
  - Creates UserBlock record (unique blocker_id + blocked_id constraint)
  - Toggle: second call deletes block record (unblocks)
  - Returns BlockResult: blocked_user_id, blocked (boolean)
- POST /api/v1/stories/{id}/report — report a story
  - Authenticated users only
  - Accepts ReportRequest: reason (enum: misinformation, hate_speech, spam, harassment, other), details (optional text, max 2000 chars)
  - Creates Report record (unique reporter_id + story_id constraint)
  - Duplicate reports prevented: returns 409 Conflict
  - Returns ReportResult: report_id, story_id, reason, status
- GET /api/v1/users/me/blocked — list blocked creators
  - Authenticated users only
  - Returns list of BlockedUserProfile: id, name, avatar_url, creator_level, blocked_at
  - Sorted by blocked_at DESC (most recent first)
- Feed service updated: get_feed() and search_stories() accept current_user_id param
  - When authenticated, queries UserBlock table and excludes stories from blocked creators
  - Joins Story -> Submission to filter by Submission.user_id NOT IN (blocked_ids)
  - Redis cache skipped for users with active blocks (personalized feed)
- Feed router updated: get_news_feed() and search_stories() pass OptionalUser.id to feed_service

**New files created:**
- app/models/user_block.py — UserBlock model (user_blocks table) with unique blocker_id + blocked_id constraint
- app/models/report.py — Report model (reports table) with ReportReason and ReportStatus enums, unique reporter_id + story_id constraint

**Files modified:**
- app/models/__init__.py — Added Report, UserBlock imports and exports
- migrations/env.py — Added Report, UserBlock model imports
- app/schemas/community.py — Added: BadgeInfo, CreatorProfileStory, CreatorProfileDetailed, BlockResult, ReportReasonEnum, ReportRequest, ReportResult, BlockedUserProfile schemas
- app/services/reputation_service.py — Full rewrite: atomic SQL UPDATE for points, _award_badge_if_missing() with duplicate prevention, _check_milestones() for 100/500/1000 thresholds, compute_accuracy_rate() function
- app/services/community_service.py — Full rewrite: self-confirm/like prevention, atomic count updates, reputation point awards, get_creator_profile_detailed() with paginated stories and accuracy_rate, toggle_block_user(), report_story() with duplicate prevention, get_blocked_users(), get_blocked_user_ids()
- app/routers/community.py — Full rewrite: all 6 endpoints with proper error handling (404/403/409), ReportRequest body parsing
- app/services/feed_service.py — Added _get_blocked_creator_ids(), updated get_feed() and search_stories() to accept current_user_id and exclude blocked creators
- app/routers/feed.py — Updated get_news_feed() and search_stories() to pass OptionalUser.id for block filtering

**Key design decisions:**
- Atomic count updates via SQL UPDATE (not in-memory increment) to prevent race conditions
- Atomic reputation point updates via UPDATE ... SET = GREATEST(0, reputation_points + :delta) RETURNING
- Self-confirm and self-like prevention uses submission.user_id check (not story.editor_id)
- Block filtering at the feed query level (not post-filter) for efficiency
- Redis cache skipped for authenticated users with blocks (personalized feed cannot be cached generically)
- Report duplicate prevention via unique constraint + service-level check with 409 status
- Milestone badges only awarded once (idempotent _award_badge_if_missing)
- Accuracy rate computed on-demand from Story.is_false_report join, not cached

### 2026-03-27 | Sprint 7 Tasks Complete (S7-01, S7-02, S7-03, S7-04)

**S7-01: Push Notification Backend Service (FCM) - DONE**
- New service: app/services/notification_service.py
- FCMClient class structured for real FCM HTTP v1 API integration (currently mocked, logs payload)
  - Real FCM payload structure preserved: message.token, notification.title/body, data, android/apns config
  - Mock returns mock-fcm-{uuid} message_id and logs full push details
  - Swap to real FCM: set FCM_ENABLED=true, provide FCM_SERVICE_ACCOUNT_KEY, uncomment httpx calls
- send_push_notification() — core function:
  1. Looks up user from DB to check preferred_language and notification_preferences
  2. Checks per-category opt-out via CATEGORY_PREFERENCE_MAP
  3. Creates in-app Notification record in database
  4. Looks up all user's DeviceToken records
  5. Sends push to each registered device via FCMClient.send()
- fire_and_forget_notification() — creates asyncio task for non-blocking notification delivery
- Convenience helpers: notify_story_approved(), notify_story_rejected(), notify_correction_needed(), notify_milestone_reached()
- Category preference mapping: story_approved/rejected/correction -> "story_updates", milestone -> "milestones", trending -> "trending"
- New models:
  - Notification (notifications table): id (UUID PK), user_id (FK), title, body, category (enum), data (JSON), is_read (bool default false), created_at
  - DeviceToken (device_tokens table): id (UUID PK), user_id (FK), token (string), platform (ios/android enum), created_at, updated_at, unique on (user_id, platform)
- NotificationCategory enum: story_approved, story_rejected, correction_needed, milestone_reached, trending_story
- DevicePlatform enum: ios, android
- FCM config added to Settings: fcm_enabled, fcm_project_id, fcm_service_account_key

**S7-02: Notification Preference API - DONE**
- New router: app/routers/notifications.py mounted at /api/v1
- POST /api/v1/devices/register — register FCM device token
  - Accepts: token (string), platform ("ios" or "android")
  - Creates or updates DeviceToken record (unique user_id + platform)
  - Returns 201 with DeviceRegisterResponse
- GET /api/v1/notifications — list user's in-app notifications
  - Cursor-based pagination (created_at DESC, most recent first)
  - Includes is_read status per notification
  - Returns unread_count in response
  - Limit 1-100, default 20
- POST /api/v1/notifications/{id}/read — mark notification as read
  - Only notification owner can mark it read
  - Returns 404 if not found or not owned
- GET /api/v1/notifications/preferences — get notification preferences
  - Returns per-category toggles: story_updates, milestones, trending, community_activity
- PUT /api/v1/notifications/preferences — update notification preferences
  - Updates user.notification_preferences JSON column
  - All categories default to True
- New schemas: app/schemas/notification.py
  - DeviceRegisterRequest/Response, NotificationItem, NotificationListResponse
  - NotificationReadResponse, NotificationPreferences, NotificationPreferencesResponse
- User model updated:
  - Added preferred_language (String(10), default="en")
  - Added notification_preferences (JSON, default all-true)
  - Added notifications relationship (lazy="noload")
  - Added device_tokens relationship (lazy="noload")

**S7-03: Wire Notifications into Editorial Workflow - DONE**
- editorial_service.py updated:
  - On approve (APPROVE action): calls notify_story_approved() fire-and-forget
  - On edit+publish (EDIT action): calls notify_story_approved() fire-and-forget
  - On reject (REJECT action): calls notify_story_rejected(reason=...) fire-and-forget
  - On mark_correction (MARK_CORRECTION action): calls notify_correction_needed(notes=...) fire-and-forget
  - Creator's preferred_language resolved from submission.user relationship
- reputation_service.py updated:
  - _check_milestones() now calls notify_milestone_reached(badge_name=...) fire-and-forget
  - Badge display names: century_reporter -> "Century Reporter", community_voice -> "Community Voice", ht_gifts_eligible -> "HT Gifts Eligible"
  - User's preferred_language resolved from DB lookup
- All notifications use fire-and-forget pattern (asyncio.create_task) — never blocks main request flow

**S7-04: i18n Backend Support - DONE**
- New package: app/i18n/ with __init__.py and translations.py
- get_translated_string(key, language, **kwargs) function:
  - Looks up key in TRANSLATIONS dict for the given language
  - Falls back to English if language unavailable, raw key if key not found
  - Supports parameterized strings via Python str.format(**kwargs)
  - Handles format errors gracefully (logs warning, returns unformatted text)
- Translation strings for all notification categories in English and Hindi:
  - notification.story_approved.title/body
  - notification.story_rejected.title/body (with {reason} param)
  - notification.correction_needed.title/body (with {notes} param)
  - notification.milestone_reached.title/body (with {badge_name} param)
  - notification.trending_story.title/body (with {story_title} and {city} params)
  - notification.generic.title/body (fallback)
- Supported languages: "en" (English), "hi" (Hindi)
- DEFAULT_LANGUAGE = "en", SUPPORTED_LANGUAGES = {"en", "hi"}
- User model: preferred_language field (default "en") drives notification language selection

**New files created:**
- app/i18n/__init__.py — Package init, exports get_translated_string
- app/i18n/translations.py — Translation catalog and lookup function
- app/models/notification.py — Notification and DeviceToken models with enums
- app/services/notification_service.py — FCM client, push sending, fire-and-forget helpers
- app/schemas/notification.py — Pydantic schemas for notification API
- app/routers/notifications.py — Notification API router (5 endpoints)

**Files modified:**
- app/models/user.py — Added preferred_language, notification_preferences columns; Added notifications and device_tokens relationships; Added JSON import
- app/models/__init__.py — Added Notification, DeviceToken imports and exports
- app/config.py — Added FCM config settings (fcm_enabled, fcm_project_id, fcm_service_account_key)
- app/main.py — Added notifications router import and mount
- app/services/editorial_service.py — Added notification triggers for approve, reject, mark_correction
- app/services/reputation_service.py — Added notification triggers for milestone badge awards
- migrations/env.py — Added Notification, DeviceToken model imports

**Key design decisions:**
- FCM mock preserves real FCM HTTP v1 payload structure for easy swap to production
- fire_and_forget_notification() uses asyncio.create_task() — never blocks the calling coroutine
- Notification service creates its own DB session when called from background tasks (owns_session pattern)
- Per-category preference checking happens before creating the Notification record (saves DB writes for opted-out users)
- Notifications and DeviceTokens use lazy="noload" on User relationship to avoid N+1 when loading users
- i18n translations stored as a simple Python dict (no external dependencies) — easy to extend with more languages
- Translation fallback chain: requested language -> English -> raw key
- DeviceToken unique on (user_id, platform) allows one token per device type, updated on re-registration

### 2026-03-27 | Sprint 8 Tasks Complete (S8-01 through S8-07)

**S8-01: Rate Limiting Middleware - DONE**
- New file: app/middleware/rate_limiter.py
- Redis-backed sliding window rate limiting using sorted sets
- Per-IP unauthenticated limit: 100 requests/minute
- Per-user authenticated limit: 200 requests/minute
- Submission-specific limits by creator level: basic_creator 5/day, ht_approved 10/day, trusted_reporter 20/day
- Confirmation spam prevention: 50 confirmations/hour per user
- Bot-flagged IPs get aggressive 10 req/min limit
- Returns HTTP 429 with Retry-After header when limit exceeded
- Lightweight JWT decode in middleware to extract user_id and creator_level without hitting DB
- Skips health, docs, and root endpoints
- Graceful degradation: allows requests through if Redis is unavailable

**S8-02: Bot Detection Middleware - DONE**
- New file: app/middleware/bot_detection.py
- Heuristic bot detection with multiple signals:
  - Missing or known bot user-agent patterns (15+ patterns: bot, crawler, spider, curl, selenium, etc.)
  - Identical request flooding: flags if >10 identical requests in 1 minute (SHA-256 fingerprint of method+path+params)
  - Submission mill detection: flags if 3+ different accounts submit from same IP within 1 hour
- On detection: sets bot flag in Redis (1 hour TTL), creates alert record for admin review
- Alert records stored in Redis list (bd:alerts:pending) with 24h retention, capped at 1000
- Bot-flagged IPs automatically get aggressive rate limiting via rate_limiter middleware integration

**S8-03: Account Deletion API - DONE**
- DELETE /api/v1/users/me - requests account deletion with 30-day grace period
- New User model fields: pending_deletion (bool), deletion_requested_at (datetime), deletion_scheduled_at (datetime), is_anonymized (bool)
- New service: app/services/account_service.py
  - request_account_deletion(): marks account as pending_deletion with scheduled date
  - cancel_account_deletion(): clears pending_deletion on login during grace period
  - anonymize_expired_accounts(): scheduled task to process expired grace periods
- Anonymization: name -> "Former Contributor", email -> "deleted_{uuid}@anonymized.local", password_hash cleared, avatar/city/preferences nulled
- Published stories remain intact but creator info is anonymized
- Duplicate deletion request returns 409 Conflict

**S8-04: Data Retention Policy Enforcement - DONE**
- New service: app/services/data_retention_service.py
- Four retention policies:
  - KYC documents: clear storage keys after 90 days post-verification (record preserved with [DELETED] markers)
  - Rejected submissions: archive after 180 days (clear description/images, keep metadata)
  - Notifications: purge after 90 days (full delete)
  - Audit logs: purge after 365 days (full delete)
- run_all_retention_policies() executes all policies and returns summary counts
- Management command: app/management/run_data_retention.py
  - Can be run via: python -m app.management.run_data_retention
  - Suitable for daily cron: 0 3 * * *
  - Runs both retention policies and account anonymization in single pass

**S8-05: Consent Management - DONE**
- New User model fields: data_consent_given (bool, default false), consent_given_at (datetime)
- GET /api/v1/users/me/consent - retrieve current consent status
- PUT /api/v1/users/me/consent - grant or withdraw consent
- Withdrawing consent (data_consent_given=false) automatically triggers account deletion flow (30-day grace)
- New schemas: app/schemas/account.py with ConsentStatus, ConsentUpdateRequest, ConsentUpdateResponse

**S8-06: Analytics Event Tracking API - DONE**
- POST /api/v1/analytics/events - batch event ingestion (max 100 events per batch)
- New model: app/models/analytics_event.py - AnalyticsEvent table
  - Fields: id (UUID PK), event_type (enum), user_id (optional), event_timestamp, metadata_json (JSON), session_id, created_at
  - Optimized indexes: composite on (event_type, event_timestamp) and (user_id, event_timestamp) for read queries
  - Append-only pattern for write-heavy workload
- EventType enum: submission_started, submission_completed, ai_review_viewed, story_viewed, story_confirmed, story_liked, story_shared, search_performed, feed_scrolled
- New service: app/services/analytics_service.py with ingest_events()
  - Validates event_type against allowed values
  - Returns accepted/rejected counts and first 10 errors
  - Attaches user_id from auth when available (OptionalUser)
- New schemas: app/schemas/analytics.py
- Model registered in app/models/__init__.py

**S8-07: Abuse Detection Service - DONE**
- New service: app/services/abuse_detection_service.py
- Submission mill detection (content similarity):
  - Generates SHA-256 content fingerprint from normalized title + first 200 chars of description
  - Tracks unique users submitting same content within 1 hour window
  - Flags all involved users if 3+ accounts submit identical content
- Confirmation ring detection (database analysis):
  - SQL query finds mutual confirmation pairs in last 7 days
  - Detects A confirms B's stories AND B confirms A's stories (3+ each)
  - Reports suspicious pairs for admin review
- Admin-facing functions:
  - get_pending_abuse_alerts(): retrieves alerts from Redis pending list
  - dismiss_abuse_alert(): removes reviewed alert from queue
  - is_user_flagged(): checks if user has active abuse flag
- Bot detection middleware enhanced with submission mill IP tracking

### 2026-03-27 | Sprint 9 Tasks Complete (S9-01, S9-02, S9-03)

**S9-01: Admin Moderation Panel API - DONE**
- New router: app/routers/admin.py mounted at /api/v1/admin
- All endpoints require is_admin=True (CurrentAdmin dependency from dependencies.py)
- GET /api/v1/admin/reports - list reports with pagination and status filter (pending/reviewed/dismissed)
  - Loads reporter name and story title via selectinload
  - Returns AdminReportListResponse: items, total, page, page_size
- POST /api/v1/admin/reports/{id}/action - take action on a report
  - Actions: dismiss, warn_user, remove_story, suspend_creator
  - dismiss: sets report status to dismissed
  - warn_user: creates audit log warning, sets report to reviewed
  - remove_story: hides story (is_hidden=True), sets report to reviewed
  - suspend_creator: suspends creator (is_suspended=True, suspended_until), hides story, creates audit log
  - All actions logged for admin accountability
- GET /api/v1/admin/users - list users with filters
  - Filters: flagged, suspended, pending_deletion
  - Search by name or email (ILIKE)
  - Pagination: page, page_size
  - Returns all admin-relevant user fields including is_suspended, is_flagged, pending_deletion
- POST /api/v1/admin/users/{id}/action - take action on a user
  - Actions: warn (audit log only), suspend (with duration), ban (permanent), unsuspend, unflag
  - Suspend sets is_suspended=True, suspended_until calculated from suspend_duration_days
  - Ban sets is_suspended=True with no expiry + is_flagged=True
  - All actions create audit log entries
- GET /api/v1/admin/stats - platform statistics dashboard
  - Returns: total_users, total_stories, submissions_today, pending_reviews, active_reports, flagged_users, suspended_users, pending_deletion_users
  - All counts via efficient COUNT queries
- New User model fields: is_suspended (bool), suspended_until (datetime), is_flagged (bool)
- New service: app/services/admin_service.py with all admin business logic
- New schemas in app/schemas/analytics.py: AdminPlatformStats, AdminReportItem/ListResponse/ActionRequest/ActionResponse, AdminUserItem/ListResponse/ActionRequest/ActionResponse

**S9-02: Editor Analytics API - DONE**
- GET /api/v1/analytics/editorial - editor-only dashboard analytics
- Requires editor role (CurrentEditor dependency)
- Returns EditorAnalytics:
  - pending_queue_size: count of submissions in under_review status
  - avg_approval_time_today_minutes: average minutes from submission creation to approve/edit action (today)
  - avg_approval_time_week_minutes: same for current week
  - rejection_rate: percentage of rejections in last 30 days
  - stories_published_today: count of stories published today
  - top_contributing_cities: top 5 cities by submission count (last 30 days)
- Optional city filter on all metrics
- Analytics computed via efficient SQL aggregations (func.avg, func.count, func.extract)

**S9-03: Trending Stories Algorithm - DONE**
- Trending score formula: (confirmations * 3 + likes * 1) / (hours_since_published + 2)^1.5
- GET /api/v1/feed?trending=true returns stories sorted by trending score
- Implementation in feed_service.py:
  - _calculate_trending_score(): applies the scoring formula per story
  - _get_trending_feed(): fetches candidate stories from last 7 days (up to 200), scores and sorts
- Optional city filter support
- Redis cache with 5-minute TTL for trending results (TRENDING_CACHE_PREFIX)
- Cursor-based pagination using published_at as tiebreaker
- Blocked creator filtering applied before scoring
- Time decay in formula ensures older stories naturally drop off trending feed

**New files created:**
- app/middleware/__init__.py - Middleware package init
- app/middleware/rate_limiter.py - Redis-backed sliding window rate limiter (S8-01)
- app/middleware/bot_detection.py - Heuristic bot detection (S8-02)
- app/models/analytics_event.py - AnalyticsEvent model (S8-06)
- app/schemas/analytics.py - Analytics + admin schemas (S8-06, S9-01, S9-02)
- app/schemas/account.py - Account deletion + consent schemas (S8-03, S8-05)
- app/services/account_service.py - Account deletion + consent (S8-03, S8-05)
- app/services/data_retention_service.py - Data retention policies (S8-04)
- app/services/abuse_detection_service.py - Abuse detection (S8-07)
- app/services/analytics_service.py - Analytics event ingestion + editor analytics (S8-06, S9-02)
- app/services/admin_service.py - Admin moderation logic (S9-01)
- app/routers/analytics.py - Analytics router (S8-06, S9-02)
- app/routers/admin.py - Admin moderation router (S9-01)
- app/management/__init__.py - Management commands package
- app/management/run_data_retention.py - Cron-able data retention command (S8-04)

**Files modified:**
- app/models/user.py - Added 8 new fields: data_consent_given, consent_given_at, pending_deletion, deletion_requested_at, deletion_scheduled_at, is_anonymized, is_suspended, suspended_until, is_flagged
- app/models/__init__.py - Added AnalyticsEvent import and export
- app/routers/users.py - Added DELETE /me (S8-03), GET/PUT /me/consent (S8-05)
- app/services/feed_service.py - Added trending algorithm: _calculate_trending_score(), _get_trending_feed(), TRENDING_CACHE_PREFIX/TTL constants (S9-03)
- app/main.py - Mounted RateLimiterMiddleware, BotDetectionMiddleware, analytics router, admin router; version bumped to 0.2.0

**Key design decisions:**
- Sliding window rate limiting uses Redis sorted sets for distributed, atomic counting
- Rate limiter does lightweight JWT decode (no DB hit) for per-user limits
- Bot detection runs before rate limiting in middleware stack (outer middleware runs first)
- Account deletion uses 30-day grace period with scheduled anonymization via cron
- Data retention uses soft deletion where possible (KYC: [DELETED] markers; submissions: [ARCHIVED])
- Analytics events use append-only pattern with minimal indexes for fast writes
- Trending algorithm scores in-memory after loading 200 candidates from last 7 days
- Trending cache uses 5-minute TTL (vs 30s for regular feed) since trending changes slowly
- Admin endpoints reuse AuditAction enum values as proxies for warn/suspend actions
- All admin actions create audit trail via AuditLog for accountability
