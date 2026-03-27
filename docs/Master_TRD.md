# Master Technical Requirements Document (TRD)
## Multi-Agent Orchestration

**Version:** 2.0.0
**Status:** Approved
**Last Updated:** 2026-03-27
**Produced by:** Root_Requirement_Agent
**Source:** PRD_Analysis.md v2.0 (38 features, 22 gaps)
**Consumed by:** Task_Breaking_Agent, Flutter_Requirement_Agent, Python_Requirement_Agent

---

## Document Conventions

- **TRD ID Format:** TRD-{MODULE}-{NNN} where MODULE is a domain abbreviation
- **Modules:** AUTH, SUB, AI, ED, FEED, COM, REP, NOTIF, PLAT, CROSS
- **Priority:** MoSCoW (Must / Should / Could / Won't)
- **State:** Approved | Pending (requires stakeholder clarification)
- **Acceptance Criteria:** Given/When/Then (Gherkin) format

---

## 1. System Overview

Mai Bhi Editor is a participatory news module for the Hindustan Times mobile app that enables citizens to submit, AI-process, editorially verify, and publish hyperlocal news.

### Architecture

```
[Flutter App] <--REST/JSON--> [FastAPI Backend] <---> [PostgreSQL]
                                    |
                              [AI Services]
                              [Object Storage]
                              [Redis Cache]
                              [FCM Push]
```

### Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Flutter (BLoC pattern, Clean Architecture) |
| Backend | Python FastAPI, SQLAlchemy ORM |
| Database | PostgreSQL 15+ |
| Cache | Redis |
| Storage | Cloud Object Storage (GCS/S3) |
| AI | LLM APIs (language processing), custom ML models (safety, duplicates) |
| Push | Firebase Cloud Messaging (FCM) |
| Search | Elasticsearch / PostgreSQL FTS |

---

# MODULE: AUTH (Authentication & Identity)

---

## TRD-AUTH-001: HT SSO Login Integration

- **Feature Ref:** F-001
- **PRD Section:** 4 (Target Users)
- **Priority:** Must
- **State:** Approved
- **Description:** Integrate with existing HT Single Sign-On system. Users authenticate via their HT credentials. The backend validates credentials against HT SSO, issues a JWT access token and refresh token pair. Authentication state persists across sessions via secure token storage on device.

**Acceptance Criteria:**

1. Given a registered HT user, When they tap "Login" and enter valid credentials, Then the backend authenticates against HT SSO and returns a JWT access token (15 min TTL) and refresh token (30 day TTL)
2. Given invalid credentials, When a user attempts login, Then an HTTP 401 response is returned with an error message and the user can retry
3. Given a logged-in user, When their access token expires, Then the app silently uses the refresh token to obtain a new access token without user interaction
4. Given a refresh token that has expired, When the app attempts silent refresh, Then the user is redirected to the login screen with a contextual message

---

## TRD-AUTH-002: JWT Token Management

- **Feature Ref:** F-001
- **PRD Section:** 4 (Target Users)
- **Priority:** Must
- **State:** Approved
- **Description:** Secure token lifecycle management. Access tokens are short-lived JWTs. Refresh tokens are stored securely (Keychain/Keystore on device, httpOnly on web). Token refresh is handled transparently by an HTTP interceptor. Logout invalidates refresh token server-side.

**Acceptance Criteria:**

1. Given a valid refresh token, When the access token expires and the app makes an API call, Then the HTTP interceptor automatically refreshes the token and retries the request
2. Given a user tapping "Logout," When the action completes, Then the refresh token is invalidated server-side and all local tokens are deleted
3. Given an unauthenticated user, When they attempt to access any protected endpoint, Then they receive HTTP 401 and are redirected to login
4. Given token storage, When tokens are persisted on device, Then they are stored in platform-secure storage (iOS Keychain / Android Keystore)

---

## TRD-AUTH-003: Creator Level Assignment

- **Feature Ref:** F-002
- **PRD Section:** 4 (Target Users)
- **Priority:** Must
- **State:** Approved
- **Description:** Three-tier creator classification system. Every authenticated user starts as "Basic Creator." Level upgrades to "HT Approved Creator" upon KYC verification, and to "Trusted Reporter" upon reaching milestone thresholds. Creator level is stored on the user profile and included in all API responses where the user is referenced.

**Acceptance Criteria:**

1. Given a new user completing first login, When their profile is created, Then creator_level is set to "basic_creator" with the corresponding badge
2. Given a Basic Creator, When their KYC verification is approved, Then creator_level upgrades to "ht_approved_creator" and badge updates propagate to all their published stories
3. Given an HT Approved Creator, When they meet the Trusted Reporter threshold (configurable, default: 500 points + 90% accuracy), Then creator_level upgrades to "trusted_reporter"
4. Given any story card or profile view, When rendered, Then the creator's current badge level is displayed next to their name

---

## TRD-AUTH-004: Creator Badge Display System

- **Feature Ref:** F-002
- **PRD Section:** 4 (Target Users)
- **Priority:** Must
- **State:** Approved
- **Description:** Visual badge system that displays the creator's level across all surfaces: story cards, detail pages, profile pages, and editor queue items. Badges are icon+label pairs (e.g., shield icon + "Trusted Reporter"). Badge assets are bundled in the app with a remote config override for new badge types.

**Acceptance Criteria:**

1. Given a story card in the feed, When rendered, Then the creator's badge icon and level name appear next to the creator name
2. Given a creator who upgrades level, When any previously published story is viewed, Then the badge reflects the current (upgraded) level
3. Given three badge levels, When the app renders any badge, Then the correct icon and color scheme is used per level (Basic=gray, Approved=blue, Trusted=gold)
4. Given badge assets, When the app loads, Then badges render from bundled assets with fallback to remote-fetched assets if updated

---

## TRD-AUTH-005: KYC Verification Flow

- **Feature Ref:** F-022
- **PRD Section:** 4 (Target Users -- mentioned but undefined)
- **Priority:** Must
- **State:** Approved
- **Description:** Document-based identity verification for "HT Approved Creator" status. Creator uploads a government-issued ID (Aadhaar, PAN, Voter ID, Passport). The system performs basic format validation, stores the document encrypted (AES-256), and places the submission in a manual KYC review queue. Admin approves or rejects with reason.

**Acceptance Criteria:**

1. Given a Basic Creator, When they navigate to KYC from their profile, Then they see a document upload form with accepted ID types listed
2. Given a valid document upload (JPEG/PNG/PDF, max 5MB), When submitted, Then the document is encrypted at rest (AES-256) and a KYC review entry is created with status "pending"
3. Given KYC approval by admin, When processed, Then the creator is upgraded to "HT Approved Creator," notified via push, and the badge updates
4. Given KYC rejection by admin, When processed, Then the creator is notified with the rejection reason and can resubmit
5. Given KYC data, When accessed by any internal user, Then the access is logged in an immutable audit trail

---

# MODULE: SUB (Submission Pipeline)

---

## TRD-SUB-001: News Submission Form API

- **Feature Ref:** F-003
- **PRD Section:** 6 (Creator Submission Flow)
- **Priority:** Must
- **State:** Approved
- **Description:** REST API for creating news submissions. Accepts mandatory fields: title (max 200 chars), cover_image_url, description (max 5000 chars), city (from allowed cities list). Optional fields: additional_image_urls[], tags[]. Returns submission ID and initial status "in_progress." Triggers the AI processing pipeline asynchronously.

**Acceptance Criteria:**

1. Given a logged-in creator with all mandatory fields, When POST /api/v1/submissions is called, Then a submission is created with status "in_progress" and the AI pipeline is triggered
2. Given missing mandatory fields, When POST /api/v1/submissions is called, Then HTTP 422 is returned with field-level validation errors
3. Given city field, When submitted, Then the city must match an entry in the allowed cities lookup table
4. Given a valid submission, When created, Then a submission_id (UUID) is returned and the creator can query status via GET /api/v1/submissions/{id}

---

## TRD-SUB-002: News Submission Form UI

- **Feature Ref:** F-003
- **PRD Section:** 6 (Creator Submission Flow)
- **Priority:** Must
- **State:** Approved
- **Description:** Flutter submission form screen with fields: title input, cover image picker (camera/gallery), description rich text area, city searchable dropdown. Inline validation on all fields. Submit button disabled until all mandatory fields are valid. Form state preserved on app backgrounding.

**Acceptance Criteria:**

1. Given a logged-in creator on the submission form, When they fill title, cover image, description, and city, Then the Submit button becomes enabled
2. Given empty mandatory fields, When the creator taps Submit, Then inline validation errors highlight each missing field
3. Given the city selector, When tapped, Then a searchable dropdown of allowed cities appears
4. Given the form with data, When the app is backgrounded or a call interrupts, Then form state is preserved in memory and restored on return
5. Given any language input, When the creator types in the description, Then the form accepts the input without language-specific validation errors

---

## TRD-SUB-003: Multi-Image Upload

- **Feature Ref:** F-004
- **PRD Section:** 6 (Creator Submission Flow)
- **Priority:** Should
- **State:** Approved
- **Description:** Support for uploading additional images beyond the mandatory cover photo. Images are uploaded to object storage, URLs stored with the submission. The UI shows thumbnail previews with reorder and delete capabilities. Published stories display images in a swipeable gallery.

**Acceptance Criteria:**

1. Given a creator on the submission form, When they tap "Add More Images," Then they can select images from gallery or camera (up to max limit)
2. Given multiple selected images, When displayed in the form, Then thumbnails show with drag-to-reorder and tap-to-delete functionality
3. Given an image exceeding size limit or unsupported format, When upload is attempted, Then a clear error message is shown per TRD-SUB-005 constraints
4. Given a published story with multiple images, When a reader opens the detail page, Then images display in a horizontally swipeable gallery

---

## TRD-SUB-004: Submission Status Lifecycle

- **Feature Ref:** F-003, F-019
- **PRD Section:** 5.B, 6 (Product Structure, Submission Flow)
- **Priority:** Must
- **State:** Approved
- **Description:** State machine for submission lifecycle: Draft -> In Progress -> AI Processing -> Under Review -> Published | Rejected | Correction Needed. Each transition is timestamped and logged. Status is queryable by the creator and visible on the dashboard.

**Acceptance Criteria:**

1. Given a new submission, When created, Then status is "in_progress" and transitions to "ai_processing" when the AI pipeline begins
2. Given AI processing completion, When all AI checks finish, Then status transitions to "under_review" and the submission enters the editor queue
3. Given an editor action (approve/reject/correct), When performed, Then status transitions to the corresponding terminal or correction state
4. Given any status transition, When it occurs, Then the transition is logged with timestamp, actor, and previous state

---

## TRD-SUB-005: Image and Media Constraints

- **Feature Ref:** F-027
- **PRD Section:** GAP
- **Priority:** Must
- **State:** Approved
- **Description:** Enforced constraints on all uploaded media. Maximum file size: 10 MB per image. Supported formats: JPEG, PNG, HEIC. Minimum resolution: 320x240. Maximum images per submission: 10 (1 cover + 9 additional). Server-side compression to web-optimized sizes for feed thumbnails and detail page display.

**Acceptance Criteria:**

1. Given an image exceeding 10 MB, When upload is attempted, Then the upload is rejected with "Image must be under 10 MB"
2. Given an unsupported format (e.g., BMP, TIFF), When upload is attempted, Then the upload is rejected with "Supported formats: JPEG, PNG, HEIC"
3. Given an image below 320x240, When uploaded, Then a warning is shown: "Image may appear low quality"
4. Given a submission with 10 images already, When the creator attempts to add another, Then the upload is blocked with "Maximum 10 images per submission"
5. Given accepted images, When stored, Then server-side compression generates: thumbnail (200px wide), feed (600px wide), and full (1200px wide) variants

---

## TRD-SUB-006: Creator Dashboard ("Posted by You")

- **Feature Ref:** F-019
- **PRD Section:** 5.B (Product Structure)
- **Priority:** Must
- **State:** Approved
- **Description:** Personal dashboard showing all of the creator's submissions with current status. Supports filtering by status (Draft, In Progress, Under Review, Published, Rejected, Correction Needed). Lists submissions in reverse chronological order. Rejected submissions show the rejection reason. Correction-needed submissions show editor comments with a resubmit option.

**Acceptance Criteria:**

1. Given a logged-in creator, When they navigate to "Posted by You," Then all their submissions are listed sorted by most recent first
2. Given the dashboard, When a status filter chip is selected, Then only submissions with that status are shown
3. Given a rejected submission, When displayed, Then the rejection reason from the editor is visible
4. Given a submission with correction request, When displayed, Then editor comments are visible and a "Revise & Resubmit" button is available
5. Given no submissions, When the dashboard loads, Then an empty state with "Write your first story" CTA is shown

---

## TRD-SUB-007: Offline Draft Support

- **Feature Ref:** F-028
- **PRD Section:** GAP
- **Priority:** Should
- **State:** Approved
- **Description:** Local draft persistence when connectivity is lost. Form state is auto-saved to local storage (SQLite/Hive) on each field change. Drafts are accessible from the "Posted by You" dashboard under "Draft" status. When connectivity returns, the creator can complete and submit. Multiple drafts are supported.

**Acceptance Criteria:**

1. Given a creator filling the form, When network connectivity is lost, Then the current form state is auto-saved as a local draft within 2 seconds
2. Given a saved draft, When the creator opens "Posted by You," Then drafts appear under "Draft" status with last-modified timestamp
3. Given a draft, When tapped, Then the submission form reopens with all previously entered data (including images) restored
4. Given multiple drafts, When listed, Then they are sorted by last-modified timestamp descending

---

# MODULE: AI (AI Processing Pipeline)

---

## TRD-AI-001: AI Language Rephrasing Service

- **Feature Ref:** F-005
- **PRD Section:** 7.1 (Language Processing)
- **Priority:** Must
- **State:** Approved
- **Description:** Backend service that takes submission text and rephrases it into editorial-quality language while preserving original meaning. Auto-detects input language. Rephrases in the same language (no translation). Uses LLM API with a system prompt tuned for news editorial style. Stores both original and rephrased text. Semantic similarity between original and rephrase must exceed 0.85.

**Acceptance Criteria:**

1. Given a submission in any supported language, When the rephrase service processes it, Then a rephrased version is generated in the same language as the input
2. Given the rephrase, When semantic similarity is computed against the original, Then the score exceeds 0.85
3. Given a Hindi input, When processed, Then the output is in Hindi (not auto-translated to English)
4. Given the AI service fails (timeout, error), When the fallback triggers, Then the original text proceeds to editor queue with a flag "AI rephrase unavailable"

---

## TRD-AI-002: AI Rephrase Preview UI

- **Feature Ref:** F-005
- **PRD Section:** 7.1 (Language Processing)
- **Priority:** Must
- **State:** Approved
- **Description:** Flutter screen showing the creator a comparison of their original text and the AI-rephrased version. Header reads "Your post -- corrected by AI." Creator can proceed (accept) but cannot edit the AI version directly. Editorial control belongs to editors, not creators.

**Acceptance Criteria:**

1. Given AI processing completes, When the creator views the preview, Then both original and AI-rephrased text are shown with clear labels
2. Given the preview screen, When the header is displayed, Then it reads "Your post -- corrected by AI"
3. Given the preview, When the creator taps "Proceed," Then the submission moves to the next pipeline stage
4. Given the preview, When displayed, Then there is no edit capability on the AI-rephrased text

---

## TRD-AI-003: AI Safety Guardrails - Content Scanning

- **Feature Ref:** F-006
- **PRD Section:** 7.2 (Guardrails and Safety)
- **Priority:** Must
- **State:** Approved
- **Description:** Automated content safety scanning service. Analyzes all submission text and images for: hate speech, toxicity, violence signals, spam, and promotional content. Produces a safety score (0.0 to 1.0, where 1.0 is safest). Content below the block threshold (configurable, default 0.2) is auto-rejected. Content between block and warning thresholds proceeds with safety warnings attached.

**Acceptance Criteria:**

1. Given a submission with hate speech, When the safety scan runs, Then the content is flagged and blocked with reason code "hate_speech" and the creator is notified
2. Given borderline content (safety score between 0.2 and 0.6), When the scan completes, Then the submission proceeds to editor queue with a visible safety warning
3. Given spam content (duplicate text, promotional links), When detected, Then the submission is auto-rejected with "Submission rejected: appears to be spam"
4. Given any submission passing safety checks, When processed, Then the safety score is stored with the submission and visible to editors

---

## TRD-AI-004: AI Safety Guardrails - Image Scanning

- **Feature Ref:** F-006
- **PRD Section:** 7.2 (Guardrails and Safety)
- **Priority:** Must
- **State:** Approved
- **Description:** Image safety analysis complementing text scanning. Scans uploaded images for explicit content, violence, and manipulated media indicators. Results contribute to the overall safety score.

**Acceptance Criteria:**

1. Given an uploaded image with explicit content, When scanned, Then the image is flagged and the submission is blocked
2. Given an image with violence indicators, When scanned, Then a warning is attached and the submission proceeds to editorial review
3. Given all images pass safety checks, When processed, Then image safety scores are recorded and contribute to the composite safety score

---

## TRD-AI-005: AI Tag and Category Suggestion

- **Feature Ref:** F-007
- **PRD Section:** 7.3 (Metadata Intelligence)
- **Priority:** Should
- **State:** Approved
- **Description:** AI analyzes submission content to suggest relevant tags (min 3) and a category from a predefined category list. Tags are presented as selectable chips. Category is auto-suggested but overridable. City field is NEVER modified by AI.

**Acceptance Criteria:**

1. Given a submission with content, When AI metadata processing runs, Then at least 3 tag suggestions are returned
2. Given AI-suggested tags, When the creator deselects a tag chip, Then it is removed from the submission metadata
3. Given AI category suggestion, When the creator selects a different category, Then the manual selection overrides the AI suggestion
4. Given the city field, When AI processing completes, Then city value is unchanged from user input

---

## TRD-AI-006: AI Duplicate Detection

- **Feature Ref:** F-008
- **PRD Section:** 7.4 (Duplicate Detection)
- **Priority:** Should
- **State:** Approved
- **Description:** Embedding-based similarity detection comparing new submissions against existing stories. Uses text embeddings combined with geo (city) and temporal (24h window) proximity signals. Similarity above threshold (configurable, default 0.8) triggers a duplicate alert. Editors see grouped duplicates with links to originals.

**Acceptance Criteria:**

1. Given a new submission about an already-reported incident (same city, within 24h, similarity > 0.8), When the duplicate scan runs, Then a "Potential Duplicate" alert is created with a link to the original story
2. Given multiple submissions about the same incident, When an editor views the queue, Then they are visually grouped as a "duplicate cluster"
3. Given a false-positive duplicate flag, When the editor dismisses it, Then the flag is removed and does not reappear
4. Given similarity below threshold, When the scan completes, Then no alert is generated

---

## TRD-AI-007: AI Editorial Confidence Score

- **Feature Ref:** F-009
- **PRD Section:** 7.5 (Editorial Confidence Score)
- **Priority:** Must
- **State:** Approved
- **Description:** Composite score (0.0 to 1.0) computed after all AI checks complete. Factors: safety score (weight 0.3), language quality score (weight 0.2), duplicate status (weight 0.1), creator reputation (weight 0.2), community confirmations (weight 0.2). Score determines default queue ordering for editors.

**Acceptance Criteria:**

1. Given a processed submission, When all AI checks complete, Then a confidence score between 0.0 and 1.0 is computed and stored
2. Given the editor review queue, When sorted by default, Then submissions are ordered by confidence score descending (highest priority first)
3. Given a submission with high safety flags, When confidence score is computed, Then safety weight pulls the score down proportionally
4. Given a submission from a Trusted Reporter, When confidence score is computed, Then the creator reputation factor positively influences the score

---

## TRD-AI-008: Multilingual AI Processing

- **Feature Ref:** F-038
- **PRD Section:** 7.1 (Language Processing)
- **Priority:** Must
- **State:** Approved
- **Description:** AI pipeline supports multiple Indian languages: Hindi, English, Marathi, Tamil, Telugu, Bengali, Kannada, Gujarati, and more. Language is auto-detected via an NLP classifier. The appropriate language model is selected for rephrasing. Quality benchmarks per language ensure rephrase quality. Unsupported languages proceed with a manual review flag.

**Acceptance Criteria:**

1. Given a submission in Tamil, When AI processes it, Then the rephrase output is in Tamil (not translated to English or Hindi)
2. Given a submission, When AI processes it, Then the detected language is recorded as metadata (language_code field)
3. Given an unsupported language, When AI encounters it, Then the submission proceeds to editor queue with flag "ai_rephrase_unavailable" and reason "unsupported_language"
4. Given AI rephrasing in any supported language, When quality-checked, Then semantic similarity with original exceeds 0.85

---

# MODULE: ED (Editorial Workflow)

---

## TRD-ED-001: Editor Review Queue API

- **Feature Ref:** F-010
- **PRD Section:** 8 (Editorial Workflow)
- **Priority:** Must
- **State:** Approved
- **Description:** API endpoint returning paginated list of submissions in "under_review" status. Each item includes: AI draft, original text, safety score, duplicate alerts, community confirmation count, confidence score, creator level, and city. Supports filtering by city and sorting by confidence score, submission time, or confirmation count.

**Acceptance Criteria:**

1. Given an editor calling GET /api/v1/editor/queue, When the queue loads, Then all "under_review" submissions are returned sorted by confidence score descending (default)
2. Given a city filter parameter, When applied, Then only submissions from that city are returned
3. Given each queue item, When returned, Then it includes: ai_draft, original_text, safety_score, duplicate_alerts[], confirmation_count, confidence_score, creator_level, city, submitted_at
4. Given pagination, When page_size=20 and cursor is provided, Then the next page of results is returned with a next_cursor

---

## TRD-ED-002: Editor Review Queue UI

- **Feature Ref:** F-010
- **PRD Section:** 8 (Editorial Workflow)
- **Priority:** Must
- **State:** Approved
- **Description:** Flutter screen for the editor review queue. Displays submission cards with key metadata. Supports city filter dropdown and sort toggle. Tapping a card opens the full review detail view. Queue auto-refreshes every 30 seconds or on pull-to-refresh.

**Acceptance Criteria:**

1. Given an editor opening the queue, When it loads, Then cards show: title, city, confidence score badge, safety indicator, confirmation count, and time since submission
2. Given the city filter, When selected, Then the queue refreshes to show only that city's submissions
3. Given the queue, When a new submission enters "under_review" status, Then the queue updates within 30 seconds (polling or push)
4. Given infinite scroll, When the editor scrolls to the bottom, Then the next page loads automatically

---

## TRD-ED-003: Editor Actions (Approve/Edit/Reject/Correct)

- **Feature Ref:** F-011
- **PRD Section:** 8 (Editorial Workflow)
- **Priority:** Must
- **State:** Approved
- **Description:** Four editor actions on submissions. Approve: publishes the story to the feed. Edit: editor modifies text/images before publishing. Reject: declines with mandatory reason text. Mark Correction: returns to creator with editor comments. Every action triggers: status update, audit log entry, push notification to creator, and analytics event.

**Acceptance Criteria:**

1. Given a submission, When editor taps "Approve," Then status changes to "published," story appears in "News By You" feed, and creator receives push notification
2. Given a submission, When editor taps "Edit," Then an inline editor opens, editor modifies content, and on save the modified version is published
3. Given a submission, When editor taps "Reject," Then a reason text field is required, status changes to "rejected," and creator is notified with the reason
4. Given a submission, When editor taps "Mark Correction," Then editor comments are required, status changes to "correction_needed," and creator is notified
5. Given any editor action, When completed, Then an audit log entry is created with: editor_id, action, timestamp, submission_id

---

## TRD-ED-004: False News Retroactive Handling

- **Feature Ref:** F-012
- **PRD Section:** 8 (Editorial Workflow)
- **Priority:** Must
- **State:** Approved
- **Description:** Post-publication misinformation handling. Editors can flag a published story as "false_report." Effects: story is unpublished or shown with correction banner, creator receives warning notification, story is removed from creator's published count, reputation points are deducted. Repeated false reports trigger creator level demotion review.

**Acceptance Criteria:**

1. Given a published story, When an editor flags it as false, Then status changes to "false_report" and the story is hidden from the feed (or shown with a correction banner)
2. Given a false-flagged story, When the creator views their dashboard, Then the story shows "False Report" status and is excluded from published count
3. Given a false-flagged story, When reputation is recalculated, Then points earned from that story are deducted
4. Given a creator with 3+ false reports, When the threshold is exceeded, Then a demotion review is triggered and flagged for admin

---

## TRD-ED-005: Content Moderation Appeal Process

- **Feature Ref:** F-023
- **PRD Section:** GAP
- **Priority:** Should
- **State:** Approved
- **Description:** Creators can appeal rejected submissions. One appeal per submission. The appeal includes creator justification text. Appeals are routed to a different editor than the original reviewer. The appeal editor can approve (proceeds to publication) or uphold rejection (final).

**Acceptance Criteria:**

1. Given a rejected submission, When the creator views it in their dashboard, Then an "Appeal" button is available (if not already appealed)
2. Given an appeal submission, When filed, Then it enters a separate appeal queue assigned to a different editor than the original reviewer
3. Given an appeal, When the reviewing editor approves, Then the story proceeds to publication and both creator and original editor are notified
4. Given an appeal, When the reviewing editor upholds the rejection, Then the rejection is final and the creator is notified with the reason
5. Given a submission already appealed, When viewed, Then the Appeal button is disabled with "Appeal already filed"

---

# MODULE: FEED (News Feed & Publishing)

---

## TRD-FEED-001: News Feed API

- **Feature Ref:** F-013
- **PRD Section:** 5.A, 9 (Publishing Experience)
- **Priority:** Must
- **State:** Approved
- **Description:** API endpoint returning paginated published citizen stories for the "News By You" feed. Default sort: recency. Supports city filter, cursor-based pagination. Each story item includes: title, cover_image_url, creator_name, creator_level, like_count, confirmation_count, city, published_at. Response time target: < 200ms p95.

**Acceptance Criteria:**

1. Given a reader calling GET /api/v1/feed, When the feed loads, Then published stories are returned sorted by published_at descending (default)
2. Given a city filter, When applied, Then only stories from that city are returned
3. Given cursor-based pagination (page_size=20), When the next cursor is provided, Then the next page loads
4. Given feed API performance, When under load, Then p95 response time is under 200ms

---

## TRD-FEED-002: News Feed UI ("News By You")

- **Feature Ref:** F-013
- **PRD Section:** 5.A (Product Structure)
- **Priority:** Must
- **State:** Approved
- **Description:** Top-level section in the HT app parallel to "News For You." Displays story cards in a scrollable list with infinite scroll. City filter at the top. Supports pull-to-refresh. Empty state for cities with no stories.

**Acceptance Criteria:**

1. Given a reader navigating to "News By You," When the section loads, Then story cards are displayed in a scrollable list
2. Given the city filter, When a city is selected, Then the feed refreshes with stories from that city only
3. Given infinite scroll, When the reader reaches the bottom, Then the next page of stories loads seamlessly
4. Given no stories for a filtered city, When displayed, Then an empty state reads "No stories yet from [City]. Be the first reporter!"

---

## TRD-FEED-003: Story Card Widget

- **Feature Ref:** F-013
- **PRD Section:** 9 (Publishing Experience)
- **Priority:** Must
- **State:** Approved
- **Description:** Reusable Flutter widget for displaying a story in the feed. Shows: title (max 2 lines), cover image (16:9 aspect), creator name + badge, like count with icon, confirmation count with icon, Confirm button (if not already confirmed), city label, and relative time.

**Acceptance Criteria:**

1. Given a story card, When rendered, Then it displays: title, cover image, creator name + badge icon, like count, confirmation count, city, and relative time
2. Given the Confirm button, When the user has not confirmed, Then the button is in "Confirm" state (tappable)
3. Given the Confirm button, When the user has already confirmed, Then the button shows "Confirmed" state (disabled)
4. Given a long title, When rendered, Then it truncates at 2 lines with ellipsis

---

## TRD-FEED-004: Story Detail Page

- **Feature Ref:** F-014
- **PRD Section:** 9 (Publishing Experience)
- **Priority:** Must
- **State:** Approved
- **Description:** Full story view with: complete content text, image gallery (swipeable), creator profile link (tappable name), editor verifier identity, "AI Verified" badge, "Editor Verified" badge, like button and count, confirmation button and count, share button. Provides full transparency on the verification chain.

**Acceptance Criteria:**

1. Given a reader tapping a story card, When the detail page loads, Then full story content, all images, and all metadata are displayed
2. Given the detail page, When rendered, Then "AI Verified" and "Editor Verified" badges are visible
3. Given the creator name, When tapped, Then it navigates to the creator's public profile (TRD-REP-002)
4. Given the editor verifier name, When displayed, Then readers can see which editor approved the story
5. Given multiple images, When displayed, Then they render in a horizontally swipeable gallery

---

## TRD-FEED-005: Trending Local Stories

- **Feature Ref:** F-020
- **PRD Section:** 5.A (Product Structure)
- **Priority:** Could
- **State:** Approved
- **Description:** Algorithm-based trending detection. Trending score = (confirmations * 3) + (likes * 1) + recency_decay_factor. Stories above the trending threshold receive a "Trending" badge in the feed. Trending is computed per city when a city filter is active. If no stories qualify, the trending section is hidden.

**Acceptance Criteria:**

1. Given the feed, When trending stories exist, Then they display with a "Trending" indicator badge
2. Given the trending algorithm, When computed, Then it weights confirmations > likes > recency
3. Given a city filter, When active, Then trending is computed only for that city's stories
4. Given insufficient engagement, When no stories meet the trending threshold, Then the trending section is hidden entirely

---

## TRD-FEED-006: Story Search

- **Feature Ref:** F-025
- **PRD Section:** GAP
- **Priority:** Should
- **State:** Approved
- **Description:** Full-text search within citizen stories. Supports keyword, creator name, city, and tag search. Results ranked by relevance (text match score) with recency as tiebreaker. Search is scoped to the "News By You" section. Backend uses PostgreSQL FTS or Elasticsearch.

**Acceptance Criteria:**

1. Given a reader in "News By You," When they enter a search query, Then matching published stories are returned ranked by relevance
2. Given search results, When displayed, Then each result uses the standard story card format
3. Given a query with no results, When displayed, Then an empty state reads "No stories found for '[query]'"
4. Given a city name as query, When searched, Then stories tagged with that city are returned

---

## TRD-FEED-007: Location-Based Feed Personalization

- **Feature Ref:** F-036
- **PRD Section:** Implied by pitch.md (Hyperlocal Coverage)
- **Priority:** Should
- **State:** Approved
- **Description:** Feed defaults to showing stories from the reader's preferred city. City preference is stored in user profile. First-time users are prompted to select their city. If location permission is granted (opt-in), device location is used to suggest the nearest city.

**Acceptance Criteria:**

1. Given a new reader, When they first open "News By You," Then they are prompted to select their city from a list
2. Given a saved city preference, When the reader opens the feed, Then stories from their preferred city are shown by default
3. Given the city selector, When changed, Then the feed immediately refreshes
4. Given location permission granted, When the reader opts in, Then device GPS suggests the nearest city

---

## TRD-FEED-008: Story Sharing

- **Feature Ref:** F-035
- **PRD Section:** GAP
- **Priority:** Should
- **State:** Approved
- **Description:** Share published stories via native OS share sheet. Share payload includes story link (deep link URI), title, and preview snippet. Deep links open the story in-app for HT app users. Non-app users see a web preview with app install CTA. Share events are tracked in analytics.

**Acceptance Criteria:**

1. Given a published story, When the user taps Share, Then the native OS share sheet opens with story title, link, and preview
2. Given a shared link, When opened by an HT app user, Then it deep-links to the story detail page in-app
3. Given a shared link, When opened by a non-app user, Then a web preview page loads with an app install CTA
4. Given a share action, When completed, Then an analytics event is fired with story_id and share_channel

---

# MODULE: COM (Community Engagement)

---

## TRD-COM-001: Community Confirmation

- **Feature Ref:** F-015
- **PRD Section:** 9, 12 (Community Layer)
- **Priority:** Must
- **State:** Approved
- **Description:** Authenticated users can confirm a story they have also witnessed. One confirmation per user per story (enforced by unique constraint). Confirmation count is displayed on story cards and detail pages. Confirmations feed into the AI editorial confidence score for related pending submissions.

**Acceptance Criteria:**

1. Given a published story, When a logged-in reader taps "Confirm," Then confirmation_count increments by 1 and the button changes to "Confirmed"
2. Given a reader who already confirmed, When they view the story, Then the button shows "Confirmed" (disabled state)
3. Given an unauthenticated user, When they tap Confirm, Then they are prompted to log in
4. Given a confirmation, When recorded, Then it contributes to the AI confidence score calculation for related pending submissions in the same city

---

## TRD-COM-002: Like Stories

- **Feature Ref:** F-016
- **PRD Section:** 9 (Publishing Experience)
- **Priority:** Should
- **State:** Approved
- **Description:** Toggle-based like system. Authenticated users can like/unlike stories. Like count is visible on feed cards and detail pages. Optimistic UI update with server sync.

**Acceptance Criteria:**

1. Given a published story, When a logged-in reader taps Like, Then like_count increments and the button shows "liked" state (optimistic update)
2. Given a liked story, When the reader taps Like again, Then the like is removed (toggle) and count decrements
3. Given an unauthenticated user, When they tap Like, Then they are prompted to log in
4. Given the like count, When displayed, Then it is visible on both the feed card and detail page

---

## TRD-COM-003: Creator Blocking and Reporting

- **Feature Ref:** F-026
- **PRD Section:** GAP
- **Priority:** Should
- **State:** Approved
- **Description:** Readers can report stories or creators for community guideline violations. Report form includes predefined reason categories (misinformation, hate speech, spam, harassment, other). Reports are queued for moderator review. Readers can block creators; blocked creators' stories are filtered from the reader's feed. Block/unblock is available from settings.

**Acceptance Criteria:**

1. Given a story or creator profile, When a reader taps "Report," Then a form appears with predefined reason categories
2. Given a submitted report, When recorded, Then it enters the moderation queue (TRD-PLAT-002) for review
3. Given a reader blocking a creator, When they browse the feed, Then that creator's stories are excluded from results
4. Given block list in settings, When a reader views it, Then they can unblock any previously blocked creator

---

# MODULE: REP (Reputation System)

---

## TRD-REP-001: Reputation Points Engine

- **Feature Ref:** F-017
- **PRD Section:** 10 (Creator Reputation System)
- **Priority:** Must
- **State:** Approved
- **Description:** Points accumulation system. Points are earned when stories are published and based on community engagement. Points are deducted for false reports. Milestones: 100 pts = badge, 1000 pts = HT Gifts eligibility, high accuracy = HT Approved Status eligibility. Points are recalculated atomically to prevent race conditions.

**Acceptance Criteria:**

1. Given a creator whose story is published, When the publish event fires, Then points are added (default: +10 per publication, configurable)
2. Given a creator reaching 100 points, When the threshold is crossed, Then a milestone badge is awarded and a push notification is sent
3. Given a creator reaching 1000 points, When the threshold is crossed, Then they are marked as eligible for HT Gifts and notified
4. Given a false news flag on a story, When confirmed, Then points earned from that story are deducted (and milestone badges may be revoked if below threshold)
5. Given concurrent events, When points are updated, Then atomic database operations prevent race conditions

**Stakeholder Clarification Needed:**
- Exact point values per action (publish, confirmation received, like received) are not defined in PRD. Using defaults: publish=10, confirmation_received=2, like_received=1. Needs product confirmation.
- "HT Gifts" are undefined. Implementation will create a "gifts_eligible" boolean flag; actual gift fulfillment is out of scope for V1.

---

## TRD-REP-002: Creator Public Profile API

- **Feature Ref:** F-018
- **PRD Section:** 11 (User Identity and Recognition)
- **Priority:** Should
- **State:** Approved
- **Description:** API endpoint returning a creator's public profile: display name, avatar URL, creator level, badge collection, published story count, reputation points (own profile only), and published story list (paginated). Accessible via GET /api/v1/creators/{id}/profile.

**Acceptance Criteria:**

1. Given a reader requesting a creator profile, When GET /api/v1/creators/{id}/profile is called, Then name, avatar, level, badges, and published_count are returned
2. Given the creator viewing their own profile, When authenticated and requesting own ID, Then reputation_points_total is also included in the response
3. Given published stories, When included in the profile, Then they are paginated and sorted by published_at descending

---

## TRD-REP-003: Creator Public Profile UI

- **Feature Ref:** F-018
- **PRD Section:** 11 (User Identity and Recognition)
- **Priority:** Should
- **State:** Approved
- **Description:** Flutter profile page showing creator's public information. Tappable from any story card. Displays: name, avatar, level badge (prominent), badge collection, published story count, and scrollable list of published stories. Own profile view adds reputation points total.

**Acceptance Criteria:**

1. Given a reader tapping a creator name, When the profile page loads, Then it shows: name, avatar, level badge, badge collection, published count, and story list
2. Given the creator viewing their own profile, When authenticated, Then reputation points total is additionally displayed
3. Given a Trusted Reporter, When profile is viewed, Then the Trusted Reporter badge is prominently displayed at the top
4. Given published stories list, When displayed, Then stories are in reverse chronological order

---

# MODULE: NOTIF (Notifications)

---

## TRD-NOTIF-001: Push Notification Service

- **Feature Ref:** F-021
- **PRD Section:** GAP
- **Priority:** Must
- **State:** Approved
- **Description:** Server-side push notification service using Firebase Cloud Messaging (FCM). Sends notifications for: story status changes (approved, rejected, correction needed), reputation milestones, community activity on creator's stories (batched), trending stories in user's city (configurable). Notifications are delivered within 60 seconds of triggering event.

**Acceptance Criteria:**

1. Given a story approval, When the editor action completes, Then the creator receives a push notification within 60 seconds
2. Given a story rejection, When the editor action completes, Then the creator receives a push with the rejection reason summary
3. Given a reputation milestone, When triggered, Then a congratulatory push is sent
4. Given community activity (likes, confirmations), When accumulated, Then a batched notification is sent (e.g., "5 people confirmed your story") -- max 1 per hour per story
5. Given notification delivery, When FCM sends, Then delivery status is logged for monitoring

---

## TRD-NOTIF-002: Notification Preferences UI

- **Feature Ref:** F-021
- **PRD Section:** GAP
- **Priority:** Must
- **State:** Approved
- **Description:** Settings screen for notification preferences. Users can enable/disable notification categories independently: story status updates, reputation milestones, community activity, trending alerts. Preferences are stored server-side and synced across devices.

**Acceptance Criteria:**

1. Given the notification settings screen, When opened, Then toggle switches for each notification category are displayed
2. Given a toggle change, When the user disables a category, Then no notifications of that type are sent to this user
3. Given preferences, When stored, Then they are persisted server-side and synced if the user logs in on another device

---

# MODULE: PLAT (Platform & Administration)

---

## TRD-PLAT-001: Admin Moderation Panel

- **Feature Ref:** F-037
- **PRD Section:** GAP
- **Priority:** Must
- **State:** Approved
- **Description:** Web-based (or in-app) admin panel for platform moderation. Queues: reported content, KYC reviews, appeal escalations, flagged accounts. Actions: warn creator, temporary suspension, permanent ban. System configuration: rate limits, AI thresholds, submission limits per level. All actions are audit-logged.

**Acceptance Criteria:**

1. Given an admin, When they open the moderation panel, Then they see queues for: reported content, KYC reviews, appeal escalations, and flagged accounts
2. Given a reported creator, When reviewed, Then the admin can issue: warning, 7-day suspension, 30-day suspension, or permanent ban
3. Given system parameters, When an admin adjusts them (e.g., daily submission limit), Then changes take effect within 60 seconds (cached config refresh)
4. Given any admin action, When performed, Then an immutable audit log entry is created

---

## TRD-PLAT-002: Editor Dashboard Analytics

- **Feature Ref:** F-034
- **PRD Section:** GAP (implied by KPIs)
- **Priority:** Could
- **State:** Approved
- **Description:** Analytics dashboard for editors showing operational metrics: pending queue size, average approval time (today/this week), rejection rate, stories published today/this week, top contributing cities, and AI accuracy metrics (false positive rates for safety flags and duplicate detection).

**Acceptance Criteria:**

1. Given an editor, When they open the dashboard, Then they see: pending count, avg. approval time, rejection rate, stories published
2. Given a city filter, When applied, Then metrics reflect only that city's data
3. Given AI accuracy metrics, When displayed, Then false positive rates for safety flags and duplicate detection are shown
4. Given the dashboard, When refreshed, Then data reflects current state (max 5 min cache)

---

## TRD-PLAT-003: Analytics and Event Tracking

- **Feature Ref:** F-029
- **PRD Section:** GAP
- **Priority:** Must
- **State:** Approved
- **Description:** Comprehensive event tracking infrastructure. Every user action fires an analytics event: submissions (funnel steps), editor actions, confirmations, likes, feed impressions, search queries, session start/end. Events include: action_type, timestamp, user_id (hashed), and contextual metadata. Data feeds dashboards for KPI monitoring.

**Acceptance Criteria:**

1. Given any user action, When it occurs, Then an analytics event is fired with action_type, timestamp, hashed_user_id, and context metadata
2. Given the submission funnel, When tracked, Then drop-off rates at each step are measurable: form_open, fields_filled, submit_tapped, ai_processed, editor_reviewed
3. Given editor actions, When tracked, Then median approval time, rejection rate, and edit rate are computable
4. Given feed engagement, When tracked, Then scroll depth, card impression rate, and click-through rate are measurable

---

## TRD-PLAT-004: Data Privacy and Retention

- **Feature Ref:** F-032
- **PRD Section:** GAP
- **Priority:** Must
- **State:** Approved
- **Description:** Data handling compliant with Indian IT Act 2000, DPDP Act, and GDPR principles. Includes: defined retention periods, user data deletion (30-day grace period), consent management, KYC data encryption (AES-256), and audit trails for all data access. Deleted accounts have stories anonymized ("Former Contributor").

**Acceptance Criteria:**

1. Given a user requesting account deletion, When initiated from settings, Then all personal data is scheduled for deletion within 30 days with a reactivation grace period
2. Given KYC documents, When stored, Then encrypted at rest with AES-256 and access is logged
3. Given data collection, When a user registers, Then explicit consent is obtained and is revocable from settings
4. Given a deleted account with published stories, When processed, Then stories are anonymized (creator name replaced with "Former Contributor")
5. Given any internal data access, When performed, Then it is logged in an immutable audit trail

---

## TRD-PLAT-005: Rate Limiting and Abuse Prevention

- **Feature Ref:** F-033
- **PRD Section:** GAP
- **Priority:** Must
- **State:** Approved
- **Description:** Multi-layered rate limiting. Per-user submission limits: Basic Creator = 5/day, HT Approved = 10/day, Trusted Reporter = 20/day. API rate limiting: 60 requests/min per user, 1000 requests/min per IP. Confirmation spam prevention: 1 confirmation per user per story. Bot detection: suspicious patterns trigger account flagging. Rate limit responses include retry-after headers.

**Acceptance Criteria:**

1. Given a Basic Creator, When they exceed 5 submissions/day, Then further submissions are blocked with "Daily limit reached. Try again tomorrow."
2. Given API requests exceeding rate, When the threshold is hit, Then HTTP 429 is returned with Retry-After header
3. Given confirmation on a story already confirmed by the user, When attempted via API, Then only the first confirmation is recorded
4. Given suspected bot activity, When patterns match (rapid submissions, identical content), Then the account is flagged for admin review

**Stakeholder Clarification Needed:**
- Exact submission limits per creator level need product confirmation. Using defaults: Basic=5, Approved=10, Trusted=20.

---

# MODULE: CROSS (Cross-Cutting Concerns)

---

## TRD-CROSS-001: Accessibility (WCAG 2.1 AA)

- **Feature Ref:** F-030
- **PRD Section:** GAP
- **Priority:** Must
- **State:** Approved
- **Description:** All UI components meet WCAG 2.1 AA standards. Requirements: color contrast ratio >= 4.5:1, screen reader compatibility (VoiceOver/TalkBack), keyboard/switch control navigation, visible focus indicators, alt text for all images, semantic widget usage in Flutter (Semantics widgets).

**Acceptance Criteria:**

1. Given any UI screen, When tested with VoiceOver (iOS) or TalkBack (Android), Then all interactive elements are properly labeled and navigable
2. Given any text/background combination, When measured, Then contrast ratio meets 4.5:1 minimum
3. Given the submission form, When navigated via switch control, Then all fields and buttons are reachable and operable
4. Given images in stories, When displayed, Then they include alt text (AI-generated description or creator-provided)
5. Given any interactive element, When focused, Then a visible focus indicator is displayed

---

## TRD-CROSS-002: Internationalization (i18n)

- **Feature Ref:** F-031
- **PRD Section:** GAP (implied by "accept any language")
- **Priority:** Should
- **State:** Approved
- **Description:** App UI supports Hindi and English at launch. All system strings, labels, error messages, and notification text are externalized in ARB files (Flutter i18n). Language is determined by device locale with manual override in settings. RTL layout support is included for future Urdu/Arabic expansion.

**Acceptance Criteria:**

1. Given a user with Hindi device locale, When the app opens, Then all UI labels and system messages appear in Hindi
2. Given the language switcher in settings, When a user selects English or Hindi, Then the UI updates immediately
3. Given error messages, When displayed, Then they appear in the user's selected language
4. Given RTL layout, When enabled (future), Then all UI elements properly mirror for right-to-left languages

---

## TRD-CROSS-003: Onboarding Flow

- **Feature Ref:** F-024
- **PRD Section:** GAP
- **Priority:** Should
- **State:** Approved
- **Description:** First-time creator experience. A 4-step tooltip walkthrough explains: how to submit a story, AI rephrase preview, editorial review process, and reputation system. Skippable at any point. Ends with mandatory acceptance of community guidelines and terms of service. Onboarding completion flag is stored and the walkthrough does not repeat for returning users.

**Acceptance Criteria:**

1. Given a user opening "News By You" for the first time, When the section loads, Then an onboarding walkthrough begins (skippable)
2. Given the walkthrough, When completed, Then the user has seen: submission process, AI rephrase preview, editorial review explanation, and reputation overview
3. Given the final step, When displayed, Then the user must accept community guidelines before first submission
4. Given a returning user, When they open "News By You," Then onboarding does not repeat

---

# Stakeholder Clarifications Required

The following items are flagged as needing product/business stakeholder input before development can proceed. Development can begin on non-dependent items.

| ID | Topic | Question | Impact | Default Assumption |
|----|-------|----------|--------|-------------------|
| CLR-001 | Story merging | Is story merging for duplicates in V1 or V2 scope? | F-008 scope | V2 (out of scope for V1) |
| CLR-002 | Creator edit after submission | Can creators edit submissions while "In Progress"? | F-003, TRD-SUB-004 | No editing after submission (only drafts are editable) |
| CLR-003 | Editor assignment model | Are editors assigned by city, language, or random? | F-010, TRD-ED-001 | Random assignment with optional city preference filter |
| CLR-004 | Confirmation geographic constraint | Limit confirmations to users in the story's city? | F-015, TRD-COM-001 | No geographic constraint (any user can confirm) |
| CLR-005 | Reputation point values | What are the point values per action? | F-017, TRD-REP-001 | publish=10, confirmation_received=2, like_received=1 |
| CLR-006 | HT Gifts definition | What are HT Gifts? Physical/digital/subscription? | F-017, TRD-REP-001 | Boolean eligibility flag; fulfillment out of V1 scope |
| CLR-007 | Creator anonymity | Is anonymous/pseudonymous reporting possible? | F-018, TRD-REP-003 | Real name only (default per PRD) |
| CLR-008 | Submission limits per level | Different rate limits per creator level? | F-033, TRD-PLAT-005 | Basic=5/day, Approved=10/day, Trusted=20/day |
| CLR-009 | Editor SLA | Target time for editorial review? | F-010, TRD-ED-001 | No enforced SLA in V1; monitored via TRD-PLAT-002 |
| CLR-010 | Web vs. Mobile | Is a web version planned? | All modules | Mobile only for V1 |

---

**END OF MASTER TRD v2.0.0**
**Status:** REQUIREMENT_APPROVED (38 features, 32 TRD specs, 10 clarifications pending)
**Next:** Task_Breaking_Agent decomposes into sprint tasks
