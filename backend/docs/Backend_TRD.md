# Backend Technical Requirements Document (Backend_TRD)
##Muti Agent- Python Backend

**Version:** 0.1.0
**Parent:** Master_TRD v0.1.0
**Last Updated:** 2026-03-27

---

## 1. Architecture

### Project Structure

```
backend/
├── app/
│   ├── main.py              # FastAPI app entry point
│   ├── config.py            # Settings (pydantic-settings)
│   ├── database.py          # Async SQLAlchemy engine + session
│   ├── dependencies.py      # Shared FastAPI dependencies
│   ├── models/              # SQLAlchemy ORM models
│   │   ├── user.py
│   │   ├── submission.py
│   │   ├── ai_review.py
│   │   ├── editor_action.py
│   │   ├── story.py
│   │   ├── confirmation.py
│   │   └── badge.py
│   ├── schemas/             # Pydantic request/response models
│   ├── routers/             # API route handlers
│   │   ├── auth.py
│   │   ├── submissions.py
│   │   ├── editorial.py
│   │   ├── feed.py
│   │   └── community.py
│   ├── services/            # Business logic layer
│   │   ├── auth_service.py
│   │   ├── submission_service.py
│   │   ├── ai_service.py
│   │   ├── editorial_service.py
│   │   ├── feed_service.py
│   │   ├── community_service.py
│   │   └── reputation_service.py
│   └── ai/                  # AI processing modules
│       ├── language.py      # Language detection + rephrasing
│       ├── safety.py        # Hate/toxicity/spam detection
│       ├── metadata.py      # Tag/category suggestion
│       ├── duplicate.py     # Duplicate detection
│       └── scoring.py       # Editorial confidence scorer
├── migrations/              # Alembic migrations
├── tests/
│   ├── unit/
│   ├── integration/
│   └── load/
├── deploy/
│   ├── cloudbuild.yaml
│   ├── Dockerfile
│   └── deployment.yaml
├── docker-compose.yaml      # Local dev (Postgres + Redis + MinIO)
├── requirements.txt
└── pyproject.toml
```

---

## 2. API Endpoints (from contract.yaml)

| Endpoint | Method | Router | TRD Ref |
|----------|--------|--------|---------|
| /auth/login | POST | auth | TRD-AUTH-001 |
| /auth/profile | GET | auth | TRD-AUTH-003 |
| /submissions | POST | submissions | TRD-SUB-001 |
| /submissions/my | GET | submissions | TRD-SUB-005 |
| /submissions/{id} | GET | submissions | TRD-SUB-004 |
| /submissions/{id}/ai-review | POST | submissions | TRD-AI-001-007 |
| /editorial/queue | GET | editorial | TRD-ED-001 |
| /editorial/{id}/action | POST | editorial | TRD-ED-003 |
| /feed | GET | feed | TRD-FEED-001 |
| /feed/{id} | GET | feed | TRD-FEED-004 |
| /stories/{id}/confirm | POST | community | TRD-COM-001 |
| /stories/{id}/like | POST | community | TRD-COM-002 |
| /creators/{id}/profile | GET | community | TRD-REP-002 |

---

## 3. Database Schema

### Core Tables

**users**
- id (UUID, PK), name, email, password_hash, avatar_url
- creator_level (enum: basic_creator, ht_approved_creator, trusted_reporter)
- reputation_points (int), city, created_at

**submissions**
- id (UUID, PK), user_id (FK), title, description, original_text
- city, status (enum: in_progress, under_review, published, rejected)
- cover_image_url, created_at, updated_at

**ai_reviews**
- id (UUID, PK), submission_id (FK)
- rewritten_text, safety_score, confidence_score
- hate_speech, toxicity, spam (booleans)
- suggested_tags (JSON), suggested_category, language_detected
- duplicate_of (UUID, nullable FK to submissions)

**editor_actions**
- id (UUID, PK), submission_id (FK), editor_id (FK to users)
- action (enum: approve, edit, reject, mark_correction)
- edited_title, edited_description, rejection_reason, notes
- created_at

**stories** (published submissions)
- id (UUID, PK), submission_id (FK), title, description
- images (JSON array), likes_count, confirmations_count
- city, published_at, editor_id (FK)
- ai_verified (bool), editor_verified (bool)

**confirmations**
- id (UUID, PK), story_id (FK), user_id (FK), created_at
- UNIQUE(story_id, user_id)

**likes**
- id (UUID, PK), story_id (FK), user_id (FK), created_at
- UNIQUE(story_id, user_id)

**badges**
- id (UUID, PK), user_id (FK), badge_type, awarded_at

---

## 4. AI Services Architecture

| Service | Input | Output | External Dependency |
|---------|-------|--------|-------------------|
| Language Detection | raw text | language code | Cloud NLP API |
| Rephrasing | raw text + lang | corrected text | Cloud NLP / LLM API |
| Safety Check | text + images | safety_score, flags | Custom model + Cloud Vision |
| Duplicate Detection | title + description | duplicate_of UUID | Vector similarity (pgvector) |
| Tag Suggestion | text | tag list | NLP classification |
| Confidence Scoring | all AI signals | confidence_score | Rule-based + ML |

---

## 5. Deployment

### Cloud Build Pipeline (cloudbuild.yaml)
1. Run tests (pytest)
2. Build Docker image
3. Push to Artifact Registry
4. Deploy to Cloud Run

### Local Development
- `docker-compose up` starts: Postgres, Redis, MinIO (S3-compatible)
- Alembic for migrations
- uvicorn with hot-reload

---

## 6. Testing Strategy

| Type | Tool | Coverage |
|------|------|----------|
| Unit tests | pytest | Services, AI modules |
| API integration | pytest + httpx | All endpoints |
| Load testing | locust | Feed API < 200ms p95 |
| DB testing | pytest + real Postgres | Migrations, queries |
