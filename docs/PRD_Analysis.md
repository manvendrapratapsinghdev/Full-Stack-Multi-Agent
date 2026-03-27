# PRD Analysis -- Mai Bhi Editor
## Produced by: PRD_Agent | Consumed by: Root_Requirement_Agent

**Version:** 2.0
**Sources:** pitch.md (Executive Vision Deck), community_pr.md (Detailed PRD)
**Date:** 2026-03-27
**Agent:** PRD_Agent (config: agents/prd_agent/config.json)

---

## Changelog

| Version | Date       | Change |
|---------|------------|--------|
| 1.0     | 2026-03-27 | Initial analysis from pitch.md + community_pr.md |
| 2.0     | 2026-03-27 | Comprehensive rewrite: full feature extraction (38 features), Given/When/Then acceptance criteria, expanded gap analysis (22 gaps), risk assessment, complete dependency graph, business goal mapping |

---

# SKILL 1: PRD Parsing -- Structured Sections

## Section 1: Vision

HT evolves from a traditional News Provider into a participatory News Community. The four-pillar model positions Citizens as first reporters, AI as assistant editor, Editors as trust guardians, and Community as the validation layer. The tagline is "From Journalism by Few to Journalism with Millions." HT becomes India's first AI-powered News Community.

**Source:** pitch.md Sections 1-3, community_pr.md Section 1

---

## Section 2: Problem Statement

Modern news consumption has shifted to creator platforms (Instagram, YouTube). Every user is a potential reporter. Speed of information exceeds newsroom reach. Trust is declining due to unverified social content.

**The Gap:**
- Traditional newsrooms: Credibility YES, Participation NO
- Social platforms: Participation YES, Credibility NO
-Muti Agentbridges both.

**Source:** pitch.md Section 1, community_pr.md Section 2

---

## Section 3: Product Goals

**Primary Goals:**
1. Increase HT app engagement and retention
2. Capture hyperlocal breaking signals first
3. Build creator ecosystem within HT
4. Expand news coverage beyond newsroom limits
5. Strengthen brand trust via transparent verification

**Business Goals:**
1. Higher session time
2. Increased DAU/MAU
3. Ad inventory growth
4. Creator loyalty
5. Community-driven organic growth

**Source:** community_pr.md Section 3

---

## Section 4: Target Users

### 4.1 Citizen Creators (Primary)
Any logged-in HT user, local eyewitness reporters, community contributors.

**Creator Levels:**

| Level               | Description                    | Unlock Criteria         |
|---------------------|--------------------------------|-------------------------|
| Basic Creator       | Any registered HT user         | Login + accept T&C      |
| HT Approved Creator | KYC-verified creator           | Document upload + review |
| Trusted Reporter    | High-accuracy veteran creator  | Milestone-based          |

### 4.2 Editors
Final verification authority. Maintain editorial standards. Moderate publication flow.

### 4.3 Readers / Community
Consume citizen news, validate reports via confirmation, participate in verification.

**Source:** community_pr.md Section 4

---

## Section 5: Product Structure

New app section "News By You" appears parallel to existing "News For You."

### 5A. Published by Others (Default Feed)
- Citizen news stories
- City filter
- Trending local stories
- Community engagement indicators (likes, confirmations)

### 5B. Posted by You (Creator Dashboard)
- Status tracking: In Progress, Under Review, Published, Rejected

**Source:** community_pr.md Section 5

---

## Section 6: Creator Submission Flow

**Mandatory Inputs:** Title, Cover Image, Description, City (user-selected)
**Optional Inputs:** Additional images, Tags (AI-suggested)

**Pipeline:** User submits --> AI Processing --> Editorial Review --> Publish/Reject

**Source:** community_pr.md Section 6

---

## Section 7: AI System Architecture

AI acts as Assistant Editor, not decision-maker.

### 7.1 Language Processing
Accept any language input. Rephrase content preserving original meaning. Show correction preview ("Your post -- corrected by AI"). Original text remains visible.

### 7.2 Guardrails and Safety
Hate speech detection, toxicity filtering, violence signals, spam detection.

### 7.3 Metadata Intelligence
AI suggests tags, category, editorial structure. City always user-controlled.

### 7.4 Duplicate Detection
Identify same incident already reported. Help editors merge signals. Future: story merging.

### 7.5 Editorial Confidence Score
AI generates review priority score for editors. Editors remain mandatory approvers.

**Source:** community_pr.md Section 7, pitch.md Section 7

---

## Section 8: Editorial Workflow

### Mandatory Human Approval
Editors receive: AI rewritten draft, original submission, safety score, duplicate alerts, community confirmations.

Editor actions: Approve, Edit, Reject, Mark Correction.

### False News Handling
If misinformation detected post-publish: warning shown to creator, story removed from totals, marked internally as false report.

**Source:** community_pr.md Section 8

---

## Section 9: Publishing Experience

### Listing Page
Displays: Title, Image, Creator Name, Badge Level, Likes, Verification Count, Confirm Button.

### Community Verification
Any user can confirm a report. Purpose: crowd validation, faster editorial awareness, community participation.

### Detail Page
Full transparency: story content, images, creator profile, editor verifier name, AI Verified badge, Editor Verified badge, likes and confirmations count.

**Source:** community_pr.md Section 9

---

## Section 10: Creator Reputation System

Gamified growth model:

| Milestone      | Reward              |
|----------------|---------------------|
| 100 points     | Badge               |
| 1000 points    | HT Gifts            |
| High Accuracy  | HT Approved Status  |

Encourages responsible reporting.

**Source:** community_pr.md Section 10

---

## Section 11: User Identity and Recognition

Creator visibility: real name (default), profile access, published history, reputation indicators. Goal: creator pride inside HT ecosystem.

**Source:** community_pr.md Section 11

---

## Section 12: Community Layer

Community as signal amplifier: validate incidents, discover hyperlocal stories, encourage participation. HT gains first breaking signals, regional engagement, local trust networks.

**Source:** community_pr.md Section 12

---

## Section 13: Business Impact

- **Engagement:** Users open HT to participate, not just read
- **Revenue:** Increased ad impressions, location-based advertising, creator-driven traffic
- **Brand:** HT becomes India's most participatory newsroom

**Source:** community_pr.md Section 13, pitch.md Sections 6, 9

---

## Section 14: KPIs / Success Metrics

1. Creator adoption rate
2. Stories submitted per day
3. Editor approval time (median)
4. Community confirmations per story
5. Engagement time (session duration)
6. Retention uplift (DAU/MAU ratio)
7. Ad revenue growth (incremental)

**Source:** community_pr.md Section 15

---

## Section 15: Long-Term Vision

Mai Bhi Editor evolves into:
- HT Creator Network
- Hyperlocal News Grid
- Early Breaking News Engine
- Independent HT Creator Platform
- AI-assisted newsroom infrastructure

**Source:** community_pr.md Section 14, pitch.md Section 10

---

# SKILL 2: Feature Extraction -- Complete Feature Cards

## F-001: User Authentication (HT SSO)
- **PRD Section:** 4 (Target Users)
- **User Role:** All
- **Description:** Users log in via existing HT SSO credentials. This is the gateway to all creator and community features. Authentication state persists across sessions with secure token management.
- **Acceptance Criteria:**
  - Given a registered HT user, When they tap "Login" and enter valid credentials, Then they receive an auth token and see the "News By You" section enabled
  - Given an unauthenticated user, When they attempt to submit news, Then they are redirected to the login screen with a contextual prompt
  - Given a logged-in user, When their session expires, Then they are silently re-authenticated or prompted to re-login without data loss
  - Given invalid credentials, When a user attempts login, Then an appropriate error message is shown and retry is allowed
- **Priority:** Must
- **Dependencies:** None (root feature)
- **Complexity:** M

---

## F-002: Creator Level System
- **PRD Section:** 4 (Target Users)
- **User Role:** Citizen Creator
- **Description:** Three-tier creator classification: Basic Creator (any HT user), HT Approved Creator (KYC verified), Trusted Reporter (earned via milestones). Each level unlocks different visibility and trust signals. Badges are displayed alongside the creator name on all published stories.
- **Acceptance Criteria:**
  - Given a new user, When they first log in, Then they are assigned "Basic Creator" level with the corresponding badge
  - Given a Basic Creator, When they complete KYC verification, Then their level upgrades to "HT Approved Creator" and badge updates across all their published stories
  - Given an HT Approved Creator, When they reach the Trusted Reporter milestone threshold, Then they receive the "Trusted Reporter" badge and a congratulatory notification
  - Given any story card, When a reader views it, Then the creator's current badge level is visible next to their name
- **Priority:** Must
- **Dependencies:** F-001
- **Complexity:** M

---

## F-003: News Submission Form
- **PRD Section:** 6 (Creator Submission Flow)
- **User Role:** Citizen Creator
- **Description:** Core submission interface allowing creators to report news with mandatory fields: title, cover image, description, and city. The form validates all inputs before allowing submission. City is always user-selected via a searchable dropdown.
- **Acceptance Criteria:**
  - Given a logged-in creator, When they fill all mandatory fields (title, cover image, description, city) and tap Submit, Then the submission enters "In Progress" status and the AI processing pipeline begins
  - Given a creator filling the form, When they leave any mandatory field empty and tap Submit, Then inline validation errors highlight the missing fields
  - Given a creator, When they select a city, Then the city is set by the user (not auto-detected) and persists through AI processing
  - Given a submission in progress, When the app is backgrounded or network drops, Then a local draft is preserved for later completion
  - Given a creator, When they write in any supported language, Then the form accepts the input without language-specific validation errors
- **Priority:** Must
- **Dependencies:** F-001
- **Complexity:** L

---

## F-004: Multi-Image Upload
- **PRD Section:** 6 (Creator Submission Flow)
- **User Role:** Citizen Creator
- **Description:** Creators can attach multiple images beyond the mandatory cover photo to provide richer context for their report. Images are displayed in a gallery view on the story detail page.
- **Acceptance Criteria:**
  - Given a creator on the submission form, When they tap "Add More Images," Then they can select additional images from their device gallery or camera
  - Given multiple images selected, When displayed in the form, Then thumbnails show with the ability to reorder or remove individual images
  - Given an image upload, When the file exceeds the size limit or is an unsupported format, Then a clear error message explains the constraint
  - Given a published story with multiple images, When a reader opens the detail page, Then images display in a swipeable gallery
- **Priority:** Should
- **Dependencies:** F-003
- **Complexity:** M

---

## F-005: AI Language Rephrasing with Preview
- **PRD Section:** 7.1 (Language Processing)
- **User Role:** Citizen Creator
- **Description:** AI rephrases the creator's submission into editorial-quality language while preserving original meaning. The creator sees a side-by-side or sequential view of their original text and the AI-corrected version. AI accepts input in any language.
- **Acceptance Criteria:**
  - Given a submitted story, When AI processing completes, Then the creator sees both original text and AI-rephrased version with a label "Your post -- corrected by AI"
  - Given a Hindi-language submission, When AI processes it, Then the rephrased version maintains Hindi (not auto-translated to English) unless the creator requests translation
  - Given AI rephrasing, When the meaning is altered beyond cosmetic correction, Then the original meaning must be preserved (testable via semantic similarity > 0.85)
  - Given the AI preview screen, When the creator reviews the rephrase, Then they can proceed to submission (they cannot edit the AI version directly -- editorial control belongs to editors)
- **Priority:** Must
- **Dependencies:** F-003
- **Complexity:** L

---

## F-006: AI Safety Guardrails
- **PRD Section:** 7.2 (Guardrails and Safety)
- **User Role:** System
- **Description:** Automated content safety layer that scans all submissions for hate speech, toxicity, violence signals, and spam. Flagged content is either blocked from the editorial queue entirely or tagged with safety warnings for editor review.
- **Acceptance Criteria:**
  - Given a submission containing hate speech, When AI safety scan runs, Then the content is flagged and blocked from the editorial queue with a reason code
  - Given a submission with borderline content, When AI safety scan runs, Then a safety score (0.0-1.0) is attached and the submission proceeds to the editor queue with a visible warning
  - Given a spam submission (duplicate text, promotional links), When AI detects it, Then the submission is auto-rejected with a user-facing explanation
  - Given any submission, When it passes safety checks, Then the safety score is recorded and visible to editors in the review queue
- **Priority:** Must
- **Dependencies:** F-003
- **Complexity:** L

---

## F-007: AI Tag and Category Suggestion
- **PRD Section:** 7.3 (Metadata Intelligence)
- **User Role:** Citizen Creator
- **Description:** AI analyzes submission content and suggests relevant tags and a category. Tags appear as selectable chips. Category is auto-suggested but can be overridden. City remains always user-controlled and is never AI-modified.
- **Acceptance Criteria:**
  - Given a submission with content, When AI metadata processing runs, Then at least 3 tag suggestions are displayed as selectable chips
  - Given AI-suggested tags, When the creator deselects a tag, Then it is removed from the submission metadata
  - Given AI category suggestion, When the creator disagrees, Then they can select a different category from a predefined list
  - Given the city field, When AI processing runs, Then the city value remains exactly as the user entered it
- **Priority:** Should
- **Dependencies:** F-003
- **Complexity:** S

---

## F-008: AI Duplicate Detection
- **PRD Section:** 7.4 (Duplicate Detection)
- **User Role:** Editor
- **Description:** AI identifies when a new submission covers the same incident as an existing report. Editors receive duplicate alerts with links to the original story, enabling them to merge signals or prioritize the most complete version.
- **Acceptance Criteria:**
  - Given a new submission about an incident already reported, When AI duplicate scan runs, Then the editor sees a "Potential Duplicate" alert with a link to the original story
  - Given multiple submissions about the same incident, When an editor views the queue, Then duplicate clusters are visually grouped or tagged
  - Given a false-positive duplicate flag, When the editor reviews, Then they can dismiss the duplicate alert and proceed with independent publication
  - Given duplicate detection, When the similarity score is below threshold, Then no alert is generated (avoiding noise)
- **Priority:** Should
- **Dependencies:** F-003, F-013
- **Complexity:** L

---

## F-009: AI Editorial Confidence Score
- **PRD Section:** 7.5 (Editorial Confidence Score)
- **User Role:** Editor
- **Description:** AI generates a composite review priority score (0.0 to 1.0) for each submission based on safety score, language quality, duplicate status, creator reputation, and community confirmations. This score determines queue ordering for editors.
- **Acceptance Criteria:**
  - Given a processed submission, When all AI checks complete, Then a confidence score between 0.0 and 1.0 is computed and stored
  - Given the editor review queue, When sorted by default, Then submissions are ordered by confidence score (highest first)
  - Given a submission with high safety flags, When the confidence score is computed, Then the score is lowered proportionally
  - Given a submission from a Trusted Reporter, When the confidence score is computed, Then creator reputation positively influences the score
- **Priority:** Must
- **Dependencies:** F-005, F-006, F-008
- **Complexity:** M

---

## F-010: Editor Review Queue
- **PRD Section:** 8 (Editorial Workflow)
- **User Role:** Editor
- **Description:** Centralized queue displaying all submissions awaiting editorial review. Each queue item shows the AI-rewritten draft, original submission, safety score, duplicate alerts, and community confirmation count. Queue is filterable by city and sortable by confidence score, time, or confirmation count.
- **Acceptance Criteria:**
  - Given an editor logging in, When they open the review queue, Then they see all pending submissions sorted by AI confidence score (default)
  - Given the queue, When an editor applies a city filter, Then only submissions from that city are displayed
  - Given a queue item, When an editor taps it, Then they see: AI draft, original text, safety score, duplicate alerts, and community confirmations side-by-side
  - Given multiple submissions, When the queue loads, Then it supports pagination or infinite scroll without performance degradation
  - Given a new submission entering the queue, When the editor has the queue open, Then the queue updates in near real-time (within 30 seconds)
- **Priority:** Must
- **Dependencies:** F-009
- **Complexity:** L

---

## F-011: Editor Actions (Approve / Edit / Reject / Correct)
- **PRD Section:** 8 (Editorial Workflow)
- **User Role:** Editor
- **Description:** Editors can take four actions on any submission: Approve (publishes the story), Edit (modify before publishing), Reject (decline with reason), Mark Correction (request creator fix). Each action triggers a status update and notification to the creator.
- **Acceptance Criteria:**
  - Given a submission in review, When an editor taps "Approve," Then the story status changes to "Published" and it appears in the "News By You" feed
  - Given a submission, When an editor taps "Edit," Then they can modify the text and images before publishing, and the published version reflects their edits
  - Given a submission, When an editor taps "Reject," Then the story status changes to "Rejected," a reason is required, and the creator is notified with the reason
  - Given a submission, When an editor taps "Mark Correction," Then the story returns to the creator with editor comments for revision
  - Given any editor action, When completed, Then the action is audit-logged with editor ID, timestamp, and action type
- **Priority:** Must
- **Dependencies:** F-010
- **Complexity:** M

---

## F-012: False News Retroactive Handling
- **PRD Section:** 8 (Editorial Workflow)
- **User Role:** Editor
- **Description:** If misinformation is detected after publication, editors can retroactively flag a story. The creator receives a warning, the story is removed from their published totals and reputation points, and the story is marked internally as a false report. The story may be unpublished or shown with a correction banner.
- **Acceptance Criteria:**
  - Given a published story, When an editor flags it as false, Then the story is marked with a "False Report" internal tag
  - Given a false-flagged story, When the creator views their dashboard, Then the story is removed from their published count and reputation points are deducted
  - Given a false-flagged story, When a reader encounters it, Then a visible correction banner or removal is shown (story should not remain live without warning)
  - Given repeated false reports from a creator, When the threshold is exceeded, Then the creator's level is subject to demotion review
- **Priority:** Must
- **Dependencies:** F-011, F-017
- **Complexity:** M

---

## F-013: News Feed ("News By You")
- **PRD Section:** 5.A, 9 (Publishing Experience)
- **User Role:** Reader
- **Description:** The primary discovery feed for citizen-reported news. Displays published stories as cards with title, image, creator name, badge level, likes, verification count, and confirm button. Supports city filter to show hyperlocal content. Appears as a top-level section parallel to "News For You."
- **Acceptance Criteria:**
  - Given a reader opening the HT app, When they navigate to "News By You," Then they see a feed of published citizen stories sorted by recency (default)
  - Given the feed, When a reader applies a city filter, Then only stories from that city are displayed
  - Given each story card, When displayed, Then it shows: title, cover image, creator name, badge level, like count, confirmation count, and a "Confirm" button
  - Given the feed, When the reader scrolls, Then infinite scroll loads additional stories without full page reload
  - Given no published stories for a city, When the reader applies that city filter, Then a meaningful empty state is shown (e.g., "No stories yet from [City]. Be the first reporter!")
- **Priority:** Must
- **Dependencies:** F-011
- **Complexity:** L

---

## F-014: Story Detail Page
- **PRD Section:** 9 (Publishing Experience)
- **User Role:** Reader
- **Description:** Full story view showing complete content, all images in a gallery, creator profile link, editor verifier identity, AI Verified badge, Editor Verified badge, like count, and confirmation count. Provides full transparency about the verification chain.
- **Acceptance Criteria:**
  - Given a reader tapping a story card, When the detail page loads, Then all story content, images, and metadata are displayed
  - Given the detail page, When rendered, Then it shows both "AI Verified" and "Editor Verified" badges
  - Given the detail page, When the creator name is tapped, Then it navigates to the creator's public profile
  - Given the detail page, When the editor verifier name is displayed, Then readers can see which editor approved the story
  - Given a story with multiple images, When the reader views the detail page, Then images are displayed in a swipeable gallery
- **Priority:** Must
- **Dependencies:** F-013
- **Complexity:** M

---

## F-015: Community Confirmation
- **PRD Section:** 9, 12 (Community Layer)
- **User Role:** Reader
- **Description:** Any authenticated user can confirm a citizen report they have also witnessed. Confirmations serve as crowd-validation signals that increase story credibility and editor awareness. The confirmation count is prominently displayed.
- **Acceptance Criteria:**
  - Given a published story, When a logged-in reader taps "Confirm," Then the confirmation count increments by 1
  - Given a reader, When they have already confirmed a story, Then the Confirm button shows a "Confirmed" state and cannot be tapped again
  - Given a story with N confirmations, When displayed on the feed or detail page, Then "Confirmed by N users" is shown
  - Given an unauthenticated user, When they tap Confirm, Then they are prompted to log in first
  - Given a confirmation, When recorded, Then it is factored into the AI editorial confidence score for related pending submissions
- **Priority:** Must
- **Dependencies:** F-013, F-001
- **Complexity:** S

---

## F-016: Like Stories
- **PRD Section:** 9 (Publishing Experience)
- **User Role:** Reader
- **Description:** Readers can like published citizen stories to signal appreciation. Like count is visible on story cards and detail pages. Likes function as a toggle (like/unlike).
- **Acceptance Criteria:**
  - Given a published story, When a logged-in reader taps the Like button, Then the like count increments and the button shows a "liked" state
  - Given a liked story, When the reader taps Like again, Then the like is removed (toggle behavior) and the count decrements
  - Given an unauthenticated user, When they tap Like, Then they are prompted to log in
  - Given the like count, When displayed, Then it is visible on both the feed card and the detail page
- **Priority:** Should
- **Dependencies:** F-013, F-001
- **Complexity:** S

---

## F-017: Creator Reputation System
- **PRD Section:** 10 (Creator Reputation System)
- **User Role:** Citizen Creator
- **Description:** Gamified points and rewards system. Creators earn points when their stories are published. Points accumulate toward milestone rewards: badges (100 pts), HT Gifts (1000 pts), and HT Approved Status (high accuracy). Points are deducted for false reports.
- **Acceptance Criteria:**
  - Given a creator whose story is approved and published, When the publication event fires, Then points are added to their reputation score
  - Given a creator reaching 100 points, When the milestone is hit, Then a badge is awarded and a notification is sent
  - Given a creator reaching 1000 points, When the milestone is hit, Then they are eligible for HT Gifts and notified
  - Given a creator with high accuracy, When they meet the threshold, Then they are eligible for upgrade to HT Approved Creator status
  - Given a false news flag on a creator's story, When the flag is confirmed, Then reputation points are deducted
- **Priority:** Must
- **Dependencies:** F-011, F-012
- **Complexity:** M

---

## F-018: Creator Public Profile
- **PRD Section:** 11 (User Identity and Recognition)
- **User Role:** Citizen Creator / Reader
- **Description:** Public-facing profile page showing creator's real name, avatar, creator level, badge collection, published story count, published history, and reputation indicators. Accessible from any story card by tapping the creator name.
- **Acceptance Criteria:**
  - Given a reader tapping a creator name on a story, When the profile loads, Then it shows: name, avatar, level, badges, published count, and story list
  - Given a creator, When they view their own profile, Then they see the same public view plus their reputation points total
  - Given a creator with Trusted Reporter status, When their profile is viewed, Then the Trusted Reporter badge is prominently displayed
  - Given a creator's published history, When displayed, Then stories are listed in reverse chronological order with status indicators
- **Priority:** Should
- **Dependencies:** F-002, F-017
- **Complexity:** M

---

## F-019: "Posted by You" Dashboard
- **PRD Section:** 5.B (Product Structure)
- **User Role:** Citizen Creator
- **Description:** Personal creator dashboard within the "News By You" section showing all the creator's submissions with their current status. Supports filtering by status: In Progress, Under Review, Published, Rejected.
- **Acceptance Criteria:**
  - Given a logged-in creator, When they navigate to "Posted by You," Then they see all their submissions sorted by most recent first
  - Given the dashboard, When a status filter is applied (e.g., "Under Review"), Then only submissions with that status are shown
  - Given a rejected submission, When displayed, Then the rejection reason from the editor is visible
  - Given a submission marked for correction, When displayed, Then the editor's correction notes are visible with an option to resubmit
  - Given no submissions, When the dashboard loads, Then a meaningful empty state with a CTA to create first story is shown
- **Priority:** Must
- **Dependencies:** F-003
- **Complexity:** M

---

## F-020: Trending Local Stories
- **PRD Section:** 5.A (Product Structure)
- **User Role:** Reader
- **Description:** Algorithmically surfaced trending stories in the feed based on a weighted combination of confirmations, likes, and recency. Trending stories receive visual emphasis (e.g., "Trending" badge or promoted position).
- **Acceptance Criteria:**
  - Given the "News By You" feed, When trending stories exist, Then they are displayed with a "Trending" indicator
  - Given the trending algorithm, When computed, Then it weights confirmations > likes > recency
  - Given a city filter is active, When trending is computed, Then only stories from that city are considered
  - Given insufficient engagement data, When no stories qualify as trending, Then the trending section is hidden (not shown empty)
- **Priority:** Could
- **Dependencies:** F-013, F-015, F-016
- **Complexity:** M

---

## F-021: Push Notification System
- **PRD Section:** GAP -- Not in original PRD
- **User Role:** All
- **Description:** System-wide push notification infrastructure to inform creators of submission status changes (approved, rejected, correction needed), milestone achievements, and community activity on their stories. Readers receive notifications for trending stories in their city.
- **Acceptance Criteria:**
  - Given a creator's story is approved, When the editor action completes, Then the creator receives a push notification within 60 seconds
  - Given a creator's story is rejected, When the editor action completes, Then the creator receives a push notification with the rejection reason summary
  - Given a creator reaches a reputation milestone, When the milestone triggers, Then they receive a congratulatory push notification
  - Given a reader with city preferences, When a story trends in their city, Then they receive a push notification (configurable)
  - Given any user, When they access notification settings, Then they can enable/disable notification categories individually
- **Priority:** Must
- **Dependencies:** F-001, F-011, F-017
- **Complexity:** L

---

## F-022: KYC Verification Flow
- **PRD Section:** 4 (Target Users -- mentioned but undefined)
- **User Role:** Citizen Creator
- **Description:** Document-based identity verification flow for creators seeking "HT Approved Creator" status. Creator uploads government-issued ID, system performs basic validation, and a manual review step confirms identity. KYC status is stored securely.
- **Acceptance Criteria:**
  - Given a Basic Creator, When they initiate KYC from their profile, Then they see a document upload form requesting government ID
  - Given a valid document upload, When processing completes, Then the submission enters a manual review queue
  - Given KYC approval, When an admin approves, Then the creator is upgraded to "HT Approved Creator" and notified
  - Given KYC rejection, When an admin rejects, Then the creator is notified with the reason and can resubmit
  - Given KYC data, When stored, Then it is encrypted at rest and access-logged per data protection requirements
- **Priority:** Must
- **Dependencies:** F-001, F-002
- **Complexity:** L

---

## F-023: Content Moderation Appeal Process
- **PRD Section:** GAP -- Not in original PRD
- **User Role:** Citizen Creator
- **Description:** Creators whose submissions are rejected can file an appeal requesting re-review. The appeal includes the creator's justification. A different editor (or senior editor) reviews the appeal to prevent bias. Limited to one appeal per submission.
- **Acceptance Criteria:**
  - Given a rejected submission, When the creator views it in their dashboard, Then an "Appeal" button is available
  - Given an appeal submission, When filed, Then it enters a separate appeal review queue assigned to a different editor than the original reviewer
  - Given an appeal, When the reviewing editor approves, Then the story proceeds to publication and the creator is notified
  - Given an appeal, When the reviewing editor upholds the rejection, Then the rejection is final and the creator is notified
  - Given a submission, When the creator has already appealed once, Then the Appeal button is disabled
- **Priority:** Should
- **Dependencies:** F-011, F-019
- **Complexity:** M

---

## F-024: Onboarding Flow for New Creators
- **PRD Section:** GAP -- Not in original PRD
- **User Role:** Citizen Creator
- **Description:** First-time creator experience that explains the submission process, AI assistance, editorial review, community validation, and reputation system. Includes a brief walkthrough or tooltip tour. Ends with acceptance of community guidelines and terms of service.
- **Acceptance Criteria:**
  - Given a user opening "News By You" for the first time, When the section loads, Then an onboarding walkthrough begins (skippable)
  - Given the walkthrough, When it completes, Then the user has seen: how to submit, AI rephrase preview, editorial review explanation, and reputation system overview
  - Given the onboarding, When the user reaches the end, Then they must accept community guidelines before their first submission
  - Given a returning user, When they open "News By You" again, Then the onboarding does not repeat
- **Priority:** Should
- **Dependencies:** F-001, F-003
- **Complexity:** S

---

## F-025: Search Functionality
- **PRD Section:** GAP -- Not in original PRD
- **User Role:** Reader
- **Description:** Search within citizen-reported stories by keyword, creator name, city, or tag. Results are ranked by relevance and recency. Search is available within the "News By You" section.
- **Acceptance Criteria:**
  - Given a reader in "News By You," When they tap the search icon and enter a keyword, Then matching stories are displayed ranked by relevance
  - Given a search query, When results are returned, Then each result shows the standard story card format
  - Given a search query with no results, When displayed, Then a meaningful empty state is shown
  - Given a search query, When the user searches by city name, Then stories tagged with that city are returned
- **Priority:** Should
- **Dependencies:** F-013
- **Complexity:** M

---

## F-026: Creator Blocking and Reporting
- **PRD Section:** GAP -- Not in original PRD
- **User Role:** Reader
- **Description:** Readers can report a story or a creator for violating community guidelines. Readers can also block a creator so their stories no longer appear in the reader's feed. Reports are queued for moderator review.
- **Acceptance Criteria:**
  - Given a story or creator profile, When a reader taps "Report," Then a report form appears with predefined reason categories
  - Given a submitted report, When recorded, Then it enters a moderation queue for review
  - Given a reader, When they block a creator, Then that creator's stories no longer appear in the reader's feed
  - Given a blocked creator, When the reader views their block list in settings, Then they can unblock previously blocked creators
- **Priority:** Should
- **Dependencies:** F-013, F-018
- **Complexity:** M

---

## F-027: Image and Media Constraints
- **PRD Section:** GAP -- Not in original PRD
- **User Role:** Citizen Creator
- **Description:** Defined constraints for uploaded media: maximum file size, supported formats, minimum resolution, and maximum number of images per submission. Images are compressed server-side to optimize storage and delivery.
- **Acceptance Criteria:**
  - Given an image upload, When the file exceeds 10 MB, Then the upload is rejected with a clear size limit message
  - Given an image upload, When the format is not JPEG, PNG, or HEIC, Then the upload is rejected with supported format guidance
  - Given an image below minimum resolution (e.g., 320x240), When uploaded, Then a warning is shown that the image may appear low quality
  - Given a submission, When the creator attempts to upload more than the maximum allowed images (e.g., 10), Then additional uploads are blocked with a limit message
  - Given accepted images, When stored, Then they are compressed to web-optimized sizes for feed and detail page delivery
- **Priority:** Must
- **Dependencies:** F-003
- **Complexity:** S

---

## F-028: Offline Draft Support
- **PRD Section:** GAP -- Not in original PRD
- **User Role:** Citizen Creator
- **Description:** When a creator is offline or has poor connectivity, their in-progress submission is saved locally as a draft. When connectivity returns, the draft can be completed and submitted. Drafts are accessible from the "Posted by You" dashboard.
- **Acceptance Criteria:**
  - Given a creator filling the submission form, When network connectivity is lost, Then the current form state is auto-saved as a local draft
  - Given a saved draft, When the creator reopens the app with connectivity, Then they see the draft in "Posted by You" under an "In Progress" / "Draft" status
  - Given a draft, When the creator taps it, Then the submission form reopens with all previously entered data restored
  - Given multiple drafts, When displayed, Then they are listed with the last-modified timestamp
- **Priority:** Should
- **Dependencies:** F-003, F-019
- **Complexity:** M

---

## F-029: Analytics and Tracking Events
- **PRD Section:** GAP -- Not in original PRD
- **User Role:** System
- **Description:** Comprehensive event tracking for all user interactions: story submissions, AI processing outcomes, editor actions, confirmations, likes, feed impressions, search queries, session duration, and funnel drop-offs. Data feeds into dashboards for KPI monitoring.
- **Acceptance Criteria:**
  - Given any user action (submit, confirm, like, search, filter), When the action occurs, Then an analytics event is fired with action type, timestamp, user ID (hashed), and contextual metadata
  - Given the submission funnel, When tracked, Then drop-off rates at each step (form open, fields filled, submit tapped, AI processed, editor reviewed) are measurable
  - Given editor actions, When tracked, Then median approval time, rejection rate, and edit rate are computable
  - Given feed engagement, When tracked, Then scroll depth, story card impression rate, and click-through rate are measurable
- **Priority:** Must
- **Dependencies:** F-001, F-003, F-013
- **Complexity:** L

---

## F-030: Accessibility (WCAG 2.1 AA Compliance)
- **PRD Section:** GAP -- Not in original PRD
- **User Role:** All
- **Description:** All UI components meet WCAG 2.1 AA standards: proper color contrast ratios, screen reader compatibility, keyboard navigation support, focus indicators, alt text for images, and semantic HTML/native component usage.
- **Acceptance Criteria:**
  - Given any UI screen, When tested with a screen reader (VoiceOver/TalkBack), Then all interactive elements are properly labeled and navigable
  - Given any text and background combination, When measured, Then color contrast ratio meets 4.5:1 minimum (AA)
  - Given the submission form, When navigated via keyboard/switch control, Then all fields and buttons are reachable and operable
  - Given uploaded images in stories, When displayed, Then they include alt text (AI-generated or creator-provided)
  - Given any interactive element, When focused, Then a visible focus indicator is displayed
- **Priority:** Must
- **Dependencies:** None (cross-cutting)
- **Complexity:** L

---

## F-031: Internationalization (i18n) Support
- **PRD Section:** GAP -- Partially implied by "accept any language"
- **User Role:** All
- **Description:** The app UI supports multiple languages (starting with Hindi and English). All system strings, labels, error messages, and notifications are translatable. RTL layout support is included for future Urdu/Arabic expansion. Content input already supports any language per PRD 7.1.
- **Acceptance Criteria:**
  - Given a user with Hindi device locale, When they open the app, Then all UI labels and system messages appear in Hindi
  - Given the language switcher, When a user selects English or Hindi, Then the UI updates to the selected language
  - Given error messages, When displayed, Then they appear in the user's selected language
  - Given notification text, When sent, Then it is in the user's preferred language
- **Priority:** Should
- **Dependencies:** None (cross-cutting)
- **Complexity:** L

---

## F-032: Data Privacy and Retention Policy
- **PRD Section:** GAP -- Not in original PRD
- **User Role:** System
- **Description:** Data handling compliant with Indian IT Act 2000 (and upcoming DPDP Act) and GDPR principles. Includes: data retention periods, user data deletion requests, consent management for data collection, KYC data encryption, and audit trails. Users can request deletion of their account and associated data.
- **Acceptance Criteria:**
  - Given a user, When they request account deletion from settings, Then all personal data is scheduled for deletion within 30 days (with a grace period for reactivation)
  - Given KYC documents, When stored, Then they are encrypted at rest with AES-256 and access is logged
  - Given user data, When collected, Then explicit consent is obtained at registration and is revocable
  - Given data retention, When the retention period expires, Then data is automatically purged per the defined schedule
  - Given any data access, When performed by internal staff, Then it is logged in an immutable audit trail
- **Priority:** Must
- **Dependencies:** F-001, F-022
- **Complexity:** L

---

## F-033: Rate Limiting and Abuse Prevention
- **PRD Section:** GAP -- Not in original PRD
- **User Role:** System
- **Description:** API rate limiting to prevent abuse: submission throttling (max N submissions per day per user), confirmation spam prevention, and bot detection. Protects AI processing pipeline from overload.
- **Acceptance Criteria:**
  - Given a creator, When they exceed the daily submission limit (e.g., 5/day for Basic Creator), Then further submissions are blocked with a clear limit message
  - Given a user, When they attempt to confirm the same story multiple times via API manipulation, Then only the first confirmation is recorded
  - Given API endpoints, When request rates exceed thresholds, Then rate limiting returns HTTP 429 with retry-after headers
  - Given suspected bot activity, When detected, Then the account is flagged for review and temporarily restricted
- **Priority:** Must
- **Dependencies:** F-001, F-003, F-015
- **Complexity:** M

---

## F-034: Editor Dashboard Analytics
- **PRD Section:** GAP -- Implied by KPIs but not detailed
- **User Role:** Editor
- **Description:** Dashboard for editors showing key operational metrics: pending queue size, average approval time, rejection rate, stories published today, top contributing cities, and AI accuracy metrics. Helps editors manage workload and spot trends.
- **Acceptance Criteria:**
  - Given an editor, When they open the dashboard, Then they see: pending count, avg. approval time (today/week), rejection rate, and stories published
  - Given the dashboard, When filtered by city, Then metrics reflect only that city's data
  - Given the dashboard, When AI accuracy metrics are shown, Then editors can see false positive rates for safety flags and duplicate detection
  - Given the dashboard, When refreshed, Then data updates to reflect current state
- **Priority:** Could
- **Dependencies:** F-010, F-011, F-029
- **Complexity:** M

---

## F-035: Story Sharing
- **PRD Section:** GAP -- Not in original PRD
- **User Role:** Reader / Citizen Creator
- **Description:** Published stories can be shared via native share sheet to external platforms (WhatsApp, Twitter/X, Instagram, etc.) and via in-app link copy. Shared links open in the HT app (deep link) or a web preview for non-app users.
- **Acceptance Criteria:**
  - Given a published story, When a user taps Share, Then the native OS share sheet appears with the story link and a preview snippet
  - Given a shared link, When opened by an HT app user, Then it deep-links to the story detail page in-app
  - Given a shared link, When opened by a non-app user, Then it opens a web preview with an app install CTA
  - Given a creator, When their story is shared, Then the share count is tracked (analytics)
- **Priority:** Should
- **Dependencies:** F-014
- **Complexity:** S

---

## F-036: Location-Based Feed Personalization
- **PRD Section:** Implied by pitch.md Section 6.1 (Hyperlocal Coverage)
- **User Role:** Reader
- **Description:** The "News By You" feed defaults to showing stories from the reader's detected or preferred city. Users can change their city preference at any time. Location detection is opt-in.
- **Acceptance Criteria:**
  - Given a new reader, When they first open "News By You," Then they are prompted to select their city (not auto-detected without consent)
  - Given a reader with a saved city preference, When they open the feed, Then stories from their preferred city are shown first
  - Given the city selector, When a reader changes their city, Then the feed immediately refreshes with stories from the new city
  - Given location permission granted, When the reader opts in, Then their device location is used to suggest the nearest city
- **Priority:** Should
- **Dependencies:** F-013
- **Complexity:** S

---

## F-037: Admin Moderation Panel
- **PRD Section:** GAP -- Not in original PRD (separate from editor queue)
- **User Role:** System (Admin)
- **Description:** Administrative panel for platform-wide moderation: reviewing reported content, managing creator bans/suspensions, reviewing KYC submissions, managing appeal escalations, and configuring system parameters (rate limits, AI thresholds).
- **Acceptance Criteria:**
  - Given an admin, When they open the moderation panel, Then they see queues for: reported content, KYC reviews, appeal escalations, and flagged accounts
  - Given a reported creator, When an admin reviews the reports, Then they can issue a warning, temporary suspension, or permanent ban
  - Given system parameters, When an admin adjusts them (e.g., daily submission limit), Then changes take effect immediately
  - Given all admin actions, When performed, Then they are logged in an immutable audit trail
- **Priority:** Must
- **Dependencies:** F-022, F-023, F-026
- **Complexity:** L

---

## F-038: Multilingual AI Processing
- **PRD Section:** 7.1 (Language Processing -- "Accept any language")
- **User Role:** System
- **Description:** AI language processing pipeline supports multiple Indian languages beyond Hindi and English: Marathi, Tamil, Telugu, Bengali, Kannada, Gujarati, and others. Language is auto-detected and the appropriate model is invoked. Rephrasing quality is validated per language.
- **Acceptance Criteria:**
  - Given a submission in Tamil, When AI processes it, Then the rephrase is in Tamil (not translated to English)
  - Given a submission, When AI processes it, Then the detected language is recorded as metadata
  - Given an unsupported language, When AI encounters it, Then the submission proceeds to editor queue with a flag indicating "AI rephrase unavailable -- manual review required"
  - Given AI rephrasing in any supported language, When quality-checked, Then semantic similarity with the original exceeds 0.85
- **Priority:** Must
- **Dependencies:** F-005
- **Complexity:** XL

---

# SKILL 3: User Story Generation

## Core User Stories

| ID     | User Story | Feature | Type |
|--------|-----------|---------|------|
| US-001 | As a citizen, I want to log in with my HT account, so that I can access the "News By You" section and submit reports | F-001 | Happy path |
| US-002 | As a citizen creator, I want to see my creator level and badges on my profile, so that I feel recognized for my contributions | F-002 | Happy path |
| US-003 | As a citizen, I want to submit a news report with a title, photo, description, and city, so that I can share what I witnessed with the HT community | F-003 | Happy path |
| US-004 | As a citizen, I want to attach multiple photos to my report, so that I can provide comprehensive visual evidence | F-004 | Happy path |
| US-005 | As a citizen, I want to see how AI corrected my text alongside my original, so that I can verify my meaning was preserved | F-005 | Happy path |
| US-006 | As an editor, I want hate speech and toxic content automatically flagged, so that I can focus my review time on legitimate reports | F-006 | Happy path |
| US-007 | As a citizen, I want AI to suggest relevant tags for my story, so that it reaches the right audience without me guessing categories | F-007 | Happy path |
| US-008 | As an editor, I want to see duplicate alerts when multiple users report the same incident, so that I can merge signals and avoid redundant publications | F-008 | Happy path |
| US-009 | As an editor, I want the review queue sorted by AI confidence score, so that I handle the most urgent and reliable stories first | F-009 | Happy path |
| US-010 | As an editor, I want to filter the review queue by city, so that I can focus on my assigned geographic area | F-010 | Happy path |
| US-011 | As an editor, I want to approve, edit, reject, or request corrections on submissions, so that I maintain HT's editorial standards | F-011 | Happy path |
| US-012 | As an editor, I want to retroactively flag a published story as false, so that misinformation is contained and the creator is held accountable | F-012 | Happy path |
| US-013 | As a reader, I want to browse citizen news stories from my city, so that I discover hyperlocal news I would not find elsewhere | F-013 | Happy path |
| US-014 | As a reader, I want to see verification badges on story detail pages, so that I know the story passed AI and editorial checks | F-014 | Happy path |
| US-015 | As a reader, I want to confirm a report I also witnessed, so that I contribute to community validation and help editors prioritize | F-015 | Happy path |
| US-016 | As a reader, I want to like stories I appreciate, so that creators get positive feedback | F-016 | Happy path |
| US-017 | As a creator, I want to earn points and badges for published stories, so that I am incentivized to report accurately and frequently | F-017 | Happy path |
| US-018 | As a reader, I want to view a creator's profile and track record, so that I can judge their credibility | F-018 | Happy path |
| US-019 | As a creator, I want a dashboard showing all my submissions and their statuses, so that I know which are pending, approved, or rejected | F-019 | Happy path |
| US-020 | As a reader, I want to see trending local stories highlighted in my feed, so that I do not miss important news in my area | F-020 | Happy path |

## Additional and Edge Case Stories

| ID     | User Story | Feature | Type |
|--------|-----------|---------|------|
| US-021 | As a creator, I want to receive a push notification when my story is approved, so that I know my contribution is live | F-021 | Happy path |
| US-022 | As a creator, I want to receive a push notification when my story is rejected (with reason), so that I understand what went wrong | F-021 | Negative path |
| US-023 | As a creator, I want to upload my government ID for KYC verification, so that I can earn "HT Approved Creator" status | F-022 | Happy path |
| US-024 | As a creator whose story was rejected, I want to appeal the decision, so that I have a fair chance at re-review | F-023 | Happy path |
| US-025 | As a first-time user, I want a brief onboarding walkthrough, so that I understand how citizen reporting works on HT | F-024 | Happy path |
| US-026 | As a reader, I want to search citizen stories by keyword or city, so that I can find specific news quickly | F-025 | Happy path |
| US-027 | As a reader, I want to report a story that appears to contain misinformation, so that moderators can review it | F-026 | Happy path |
| US-028 | As a reader, I want to block a creator whose content I find inappropriate, so that their stories no longer appear in my feed | F-026 | Happy path |
| US-029 | As a creator, I want clear error messages when my image upload fails (too large, wrong format), so that I can fix and retry | F-027 | Negative path |
| US-030 | As a creator with poor connectivity, I want my in-progress submission saved as a draft, so that I do not lose my work | F-028 | Edge case |
| US-031 | As a product manager, I want to track submission funnel drop-offs, so that I can identify and fix UX bottlenecks | F-029 | System |
| US-032 | As a visually impaired user, I want all buttons and images to have proper labels, so that I can use the app with a screen reader | F-030 | Accessibility |
| US-033 | As a Hindi-speaking user, I want the app UI in Hindi, so that I can navigate without language barriers | F-031 | i18n |
| US-034 | As a user, I want to delete my account and all associated data, so that my privacy is protected | F-032 | Privacy |
| US-035 | As a system, I want to throttle excessive submissions from a single user, so that abuse is prevented | F-033 | System |
| US-036 | As an editor, I want to see my operational metrics (approval time, rejection rate), so that I can improve my workflow | F-034 | Happy path |
| US-037 | As a creator, I want to share my published story on WhatsApp, so that my friends and family can read it | F-035 | Happy path |
| US-038 | As a reader, I want the feed to default to my city, so that I see relevant local news immediately | F-036 | Happy path |
| US-039 | As an admin, I want to suspend a creator who repeatedly submits false reports, so that platform integrity is maintained | F-037 | Happy path |
| US-040 | As a creator writing in Marathi, I want AI to rephrase my content in Marathi (not English), so that my story retains its linguistic identity | F-038 | Happy path |
| US-041 | As a creator, I want to save a partially filled form and come back later, so that I can report news at my convenience | F-028 | Happy path |
| US-042 | As a reader, I want to see "No stories yet from [City]" when no content exists for my filter, so that I am not confused by an empty screen | F-013 | Edge case (empty state) |
| US-043 | As a creator, I want to know why my appeal was upheld or overturned, so that I can adjust my future submissions | F-023 | Negative path |
| US-044 | As an editor, I want to see which submissions have high community confirmation counts, so that I can fast-track likely-valid stories | F-010 | Happy path |
| US-045 | As a creator, I want to be warned if I am about to exceed my daily submission limit, so that I prioritize my most important report | F-033 | Edge case |

---

# SKILL 4: Gap Analysis

## Comprehensive Gap Register

| Gap ID   | Gap Description | Severity | Category | Addressed by Feature | Recommendation |
|----------|----------------|----------|----------|---------------------|----------------|
| GAP-001  | No offline submission behavior defined | Medium | Offline | F-028 | Queue submissions locally as drafts, sync on reconnect, show draft status in dashboard |
| GAP-002  | No image size/format constraints specified | Medium | Media | F-027 | Define max 10 MB per image, supported formats: JPEG/PNG/HEIC, max 10 images per submission, minimum resolution 320x240 |
| GAP-003  | No notification system described anywhere in PRD | High | Notifications | F-021 | Full push notification system for: story status changes, reputation milestones, trending stories in user's city, community activity |
| GAP-004  | KYC verification flow mentioned but not detailed | High | Identity | F-022 | Define document upload UI, supported ID types, manual review process, approval/rejection flow, data security requirements |
| GAP-005  | Multi-language input accepted but no spec on supported languages | Medium | i18n | F-031, F-038 | Start with Hindi + English for UI, support 8+ Indian languages for AI processing, define fallback for unsupported languages |
| GAP-006  | No content moderation appeal process for rejected stories | High | Moderation | F-023 | Allow one appeal per rejection, route to different editor, provide reason transparency |
| GAP-007  | No data retention or deletion policy | High | Privacy/Legal | F-032 | Define retention periods per data type, DPDP Act / GDPR compliance, user deletion rights, KYC data handling |
| GAP-008  | Accessibility requirements not addressed | High | Accessibility | F-030 | Target WCAG 2.1 AA: color contrast, screen reader support, keyboard navigation, focus indicators, alt text |
| GAP-009  | No search functionality within citizen news | Medium | Discovery | F-025 | Keyword search, city search, tag search, creator name search within "News By You" |
| GAP-010  | No creator blocking or reporting mechanism | High | Safety | F-026 | Report story/creator with predefined reasons, block creator from personal feed, moderation queue for reports |
| GAP-011  | No onboarding flow for first-time creators | Medium | UX | F-024 | Tooltip tour or brief walkthrough explaining submission process, AI assistance, editorial review, and reputation system |
| GAP-012  | No analytics or event tracking specification | High | Product Ops | F-029 | Define tracking events for entire submission funnel, engagement actions, editor workflow, and business KPIs |
| GAP-013  | No rate limiting or abuse prevention | High | Security | F-033 | Daily submission limits per creator level, confirmation spam prevention, API rate limiting, bot detection |
| GAP-014  | No story sharing capability | Medium | Growth | F-035 | Native share sheet, deep linking for app users, web preview for non-app users |
| GAP-015  | No location-based feed personalization | Medium | UX | F-036 | Default feed to user's preferred city, opt-in location detection, easy city switching |
| GAP-016  | No admin moderation panel (separate from editor queue) | High | Operations | F-037 | Platform-wide moderation: reported content queue, creator bans, KYC admin, appeal escalations, system config |
| GAP-017  | No editor performance dashboard | Low | Operations | F-034 | Operational metrics for editors: queue size, avg. approval time, rejection rate, AI accuracy |
| GAP-018  | No error state definitions for AI processing failures | Medium | Reliability | -- | Define fallback when AI rephrase fails (proceed with original text + flag), when safety check times out (queue for manual review), when duplicate detection is unavailable |
| GAP-019  | No content expiration or archival policy | Low | Content Ops | -- | Define whether citizen stories expire, are archived after N days, or persist indefinitely. Recommend 1-year active window, then archive |
| GAP-020  | No API versioning or backwards compatibility strategy | Low | Technical | -- | Define API versioning strategy (URL-based or header-based) for mobile client compatibility across app versions |
| GAP-021  | No story editing by creator after submission | Medium | UX | -- | PRD does not specify if creators can edit submissions before editorial review. Recommend allowing edits while status is "In Progress" only |
| GAP-022  | No specification of what happens to stories when a creator deletes their account | Medium | Privacy | F-032 | Define: anonymize stories (replace creator name with "Former Contributor") or delete stories entirely. Recommend anonymization to preserve community value |

---

# SKILL 5: Business Goal Mapping

## Feature-to-KPI Traceability Matrix

### KPI 1: Creator Adoption Rate
Measures how many HT users become active creators.

| Feature | Contribution |
|---------|-------------|
| F-001 (Auth) | Gateway to creator role |
| F-002 (Creator Levels) | Aspirational progression encourages sign-up |
| F-003 (Submission) | Core creation action |
| F-017 (Reputation) | Gamification drives adoption |
| F-018 (Profile) | Public recognition motivates new creators |
| F-022 (KYC) | Pathway to elevated status |
| F-024 (Onboarding) | Reduces friction for first-time creators |
| F-031 (i18n) | Removes language barriers for non-English users |

### KPI 2: Stories Submitted per Day
Measures content creation velocity.

| Feature | Contribution |
|---------|-------------|
| F-003 (Submission) | Core submission mechanism |
| F-004 (Multi-Image) | Richer submissions encourage more reports |
| F-005 (AI Rephrase) | Lowers quality bar for input, AI elevates output |
| F-007 (AI Tags) | Reduces friction in metadata entry |
| F-019 (Dashboard) | Status visibility encourages resubmission |
| F-028 (Offline Drafts) | Prevents lost submissions due to connectivity |
| F-038 (Multilingual AI) | Enables creators in regional languages |

### KPI 3: Editor Approval Time (Median)
Measures editorial workflow efficiency.

| Feature | Contribution |
|---------|-------------|
| F-006 (Safety) | Auto-filters unsafe content, reduces noise |
| F-008 (Duplicates) | Reduces redundant reviews |
| F-009 (Confidence) | Prioritizes queue for efficiency |
| F-010 (Queue) | Organized workspace for editors |
| F-011 (Actions) | Streamlined approve/reject/edit workflow |
| F-034 (Editor Analytics) | Self-monitoring drives improvement |

### KPI 4: Community Confirmations per Story
Measures community validation engagement.

| Feature | Contribution |
|---------|-------------|
| F-015 (Confirm) | Direct confirmation mechanism |
| F-013 (Feed) | Discovery surface for confirmable stories |
| F-014 (Detail) | Detailed view encourages informed confirmation |
| F-021 (Notifications) | Alerts users about stories in their area |
| F-036 (Location Feed) | Surfaces relevant stories users can confirm |

### KPI 5: Engagement Time (Session Duration)
Measures time spent in app.

| Feature | Contribution |
|---------|-------------|
| F-013 (Feed) | Scrollable content increases session time |
| F-014 (Detail) | Deep reading increases time on page |
| F-016 (Like) | Engagement actions extend sessions |
| F-020 (Trending) | Discovery of popular content |
| F-025 (Search) | Active content seeking extends sessions |
| F-035 (Sharing) | Social loop drives return visits |

### KPI 6: Retention Uplift (DAU/MAU Ratio)
Measures daily return rate.

| Feature | Contribution |
|---------|-------------|
| F-002 (Creator Levels) | Progression motivates daily return |
| F-017 (Reputation) | Points accumulation drives daily use |
| F-019 (Dashboard) | Status checking drives return visits |
| F-021 (Notifications) | Push notifications drive re-engagement |
| F-036 (Location Feed) | Relevant local content creates habit |

### KPI 7: Ad Revenue Growth (Incremental)
Measures monetization uplift from the new section.

| Feature | Contribution |
|---------|-------------|
| F-013 (Feed) | New ad inventory surface (in-feed ads) |
| F-014 (Detail) | New ad inventory surface (in-article ads) |
| F-020 (Trending) | High-traffic surface for premium ads |
| F-036 (Location Feed) | Location-based ad targeting |
| F-029 (Analytics) | Data for ad targeting and reporting |

---

# SKILL 6: Dependency Graph

## Full Dependency Tree

```
TIER 0 -- FOUNDATION (No dependencies)
  F-001 (User Authentication)
  F-030 (Accessibility) -- cross-cutting
  F-031 (i18n) -- cross-cutting

TIER 1 -- CORE CREATION (Depends on F-001)
  F-002 (Creator Levels) --> F-001
  F-003 (News Submission) --> F-001
  F-022 (KYC Flow) --> F-001, F-002
  F-024 (Onboarding) --> F-001, F-003

TIER 2 -- AI PROCESSING (Depends on F-003)
  F-004 (Multi-Image) --> F-003
  F-005 (AI Rephrase) --> F-003
  F-006 (AI Safety) --> F-003
  F-007 (AI Tags) --> F-003
  F-008 (AI Duplicates) --> F-003, F-013
  F-019 (Dashboard) --> F-003
  F-027 (Image Constraints) --> F-003
  F-028 (Offline Drafts) --> F-003, F-019
  F-033 (Rate Limiting) --> F-001, F-003, F-015
  F-038 (Multilingual AI) --> F-005

TIER 3 -- EDITORIAL (Depends on AI tier)
  F-009 (Confidence Score) --> F-005, F-006, F-008
  F-010 (Editor Queue) --> F-009
  F-011 (Editor Actions) --> F-010
  F-023 (Appeal Process) --> F-011, F-019

TIER 4 -- PUBLISHING (Depends on Editor tier)
  F-012 (False News) --> F-011, F-017
  F-013 (News Feed) --> F-011
  F-017 (Reputation) --> F-011, F-012
  F-021 (Notifications) --> F-001, F-011, F-017
  F-029 (Analytics) --> F-001, F-003, F-013

TIER 5 -- CONSUMPTION (Depends on Publishing tier)
  F-014 (Story Detail) --> F-013
  F-015 (Confirm) --> F-013, F-001
  F-016 (Like) --> F-013, F-001
  F-018 (Creator Profile) --> F-002, F-017
  F-025 (Search) --> F-013
  F-026 (Block/Report) --> F-013, F-018
  F-035 (Sharing) --> F-014
  F-036 (Location Feed) --> F-013

TIER 6 -- ADVANCED (Depends on Consumption tier)
  F-020 (Trending) --> F-013, F-015, F-016
  F-034 (Editor Analytics) --> F-010, F-011, F-029
  F-037 (Admin Panel) --> F-022, F-023, F-026
  F-032 (Data Privacy) --> F-001, F-022
```

## Critical Path (Longest chain)

```
F-001 --> F-003 --> F-005 --> F-009 --> F-010 --> F-011 --> F-013 --> F-014
```

This 8-feature chain is the minimum path to a publishable, viewable citizen story. All sprint planning must prioritize this chain.

## Parallel Tracks (Can be built simultaneously)

- **Track A (AI Pipeline):** F-005, F-006, F-007, F-008 -- all depend on F-003, independent of each other
- **Track B (Consumer Experience):** F-015, F-016, F-025, F-036 -- all depend on F-013, independent of each other
- **Track C (Creator Experience):** F-019, F-028 -- depend on F-003, independent of editorial chain
- **Track D (Cross-cutting):** F-030, F-031 -- no feature dependencies, can start anytime

---

# SKILL 7: Risk Assessment

## Technical Risks

| Risk ID | Risk | Probability | Impact | Mitigation |
|---------|------|-------------|--------|------------|
| TR-001 | AI rephrase alters meaning of sensitive reports (e.g., casualty numbers, political statements) | High | High | Semantic similarity threshold (>0.85), mandatory human preview, editor sees both versions |
| TR-002 | Multilingual AI quality inconsistent across Indian languages | High | Medium | Start with Hindi+English, expand incrementally, per-language quality benchmarks before launch |
| TR-003 | AI safety guardrails produce excessive false positives, frustrating legitimate creators | Medium | High | Tunable thresholds, editor override for borderline cases, track false positive rate, creator feedback loop |
| TR-004 | Duplicate detection fails for semantically similar but textually different reports | Medium | Medium | Use embedding-based similarity (not just text matching), combine with geo+time proximity signals |
| TR-005 | High submission volume overwhelms editorial capacity | High | High | AI confidence scoring to prioritize, auto-reject clear safety violations, scale editor team based on submission/editor ratio metrics |
| TR-006 | Image upload latency on slow mobile networks (2G/3G prevalent in hyperlocal areas) | High | Medium | Client-side compression before upload, progressive upload with retry, offline draft support |
| TR-007 | Single point of failure in AI processing pipeline blocks all submissions | Medium | High | Circuit breaker pattern, fallback to manual-only review when AI is down, queue submissions for retry |
| TR-008 | Push notification deliverability across fragmented Android ecosystem | Medium | Medium | Use FCM with fallback polling, track delivery rates per device segment, degrade gracefully |

## Business Risks

| Risk ID | Risk | Probability | Impact | Mitigation |
|---------|------|-------------|--------|------------|
| BR-001 | Low creator adoption -- users do not transition from consumers to contributors | Medium | High | Gamification (F-017), onboarding (F-024), prominent "News By You" placement, creator success stories, initial seeding with influencer partnerships |
| BR-002 | Misinformation incident damages HT brand trust | Medium | Critical | Mandatory editor approval (F-011), false news handling (F-012), AI safety (F-006), community confirmation as signal (not authority) |
| BR-003 | Editor bottleneck slows publication, frustrating creators | High | High | AI confidence scoring (F-009), tiered review (Trusted Reporters get fast-track), scale editorial ops, set SLA targets |
| BR-004 | Competitor launches similar product (TOI, NDTV) | Medium | Medium | First mover advantage, deep integration with HT ecosystem, superior AI pipeline, creator lock-in via reputation system |
| BR-005 | Legal liability for user-generated content | Medium | High | Clear terms of service, editorial approval as legal checkpoint, rapid takedown process, indemnification clauses, legal review of platform liability under IT Act |
| BR-006 | Creator fatigue from low approval rate or slow feedback | Medium | High | Transparent status tracking (F-019), push notifications (F-021), target <2hr median approval time, editor capacity planning |
| BR-007 | Gaming of reputation system (fake confirmations, submission mills) | Medium | Medium | Rate limiting (F-033), bot detection, geo-fencing confirmations to story city, anomaly detection on confirmation patterns |
| BR-008 | Regional language content creates editorial blind spots (editors cannot review languages they don't speak) | High | Medium | Language-specific editor assignment, AI translation for editor preview (clearly marked), hire multilingual editors for top regional languages |

## Operational Risks

| Risk ID | Risk | Probability | Impact | Mitigation |
|---------|------|-------------|--------|------------|
| OR-001 | Insufficient editorial staff to handle submission volume at scale | High | High | Phase launches city-by-city, monitor submission/editor ratio, hire proactively based on growth curve |
| OR-002 | AI model costs exceed budget at scale | Medium | Medium | Batch processing where possible, cache duplicate detection results, optimize model selection (smaller models for simpler tasks) |
| OR-003 | KYC document handling creates DPDP Act compliance burden | Medium | High | Minimize KYC data retention (delete after verification), encrypt at rest, engage legal for compliance review |

---

# Appendix A: Feature Summary Table

| FID   | Title                          | Priority | Complexity | Tier | Dependencies |
|-------|-------------------------------|----------|------------|------|--------------|
| F-001 | User Authentication            | Must     | M          | 0    | None |
| F-002 | Creator Level System           | Must     | M          | 1    | F-001 |
| F-003 | News Submission Form           | Must     | L          | 1    | F-001 |
| F-004 | Multi-Image Upload             | Should   | M          | 2    | F-003 |
| F-005 | AI Language Rephrasing         | Must     | L          | 2    | F-003 |
| F-006 | AI Safety Guardrails           | Must     | L          | 2    | F-003 |
| F-007 | AI Tag/Category Suggestion     | Should   | S          | 2    | F-003 |
| F-008 | AI Duplicate Detection         | Should   | L          | 2    | F-003, F-013 |
| F-009 | AI Editorial Confidence Score  | Must     | M          | 3    | F-005, F-006, F-008 |
| F-010 | Editor Review Queue            | Must     | L          | 3    | F-009 |
| F-011 | Editor Actions                 | Must     | M          | 3    | F-010 |
| F-012 | False News Handling            | Must     | M          | 4    | F-011, F-017 |
| F-013 | News Feed                      | Must     | L          | 4    | F-011 |
| F-014 | Story Detail Page              | Must     | M          | 5    | F-013 |
| F-015 | Community Confirmation         | Must     | S          | 5    | F-013, F-001 |
| F-016 | Like Stories                   | Should   | S          | 5    | F-013, F-001 |
| F-017 | Creator Reputation System      | Must     | M          | 4    | F-011, F-012 |
| F-018 | Creator Public Profile         | Should   | M          | 5    | F-002, F-017 |
| F-019 | Posted by You Dashboard        | Must     | M          | 2    | F-003 |
| F-020 | Trending Local Stories         | Could    | M          | 6    | F-013, F-015, F-016 |
| F-021 | Push Notification System       | Must     | L          | 4    | F-001, F-011, F-017 |
| F-022 | KYC Verification Flow          | Must     | L          | 1    | F-001, F-002 |
| F-023 | Content Moderation Appeal      | Should   | M          | 3    | F-011, F-019 |
| F-024 | Onboarding Flow                | Should   | S          | 1    | F-001, F-003 |
| F-025 | Search Functionality           | Should   | M          | 5    | F-013 |
| F-026 | Creator Blocking/Reporting     | Should   | M          | 5    | F-013, F-018 |
| F-027 | Image/Media Constraints        | Must     | S          | 2    | F-003 |
| F-028 | Offline Draft Support          | Should   | M          | 2    | F-003, F-019 |
| F-029 | Analytics and Tracking         | Must     | L          | 4    | F-001, F-003, F-013 |
| F-030 | Accessibility (WCAG 2.1 AA)   | Must     | L          | 0    | None |
| F-031 | Internationalization (i18n)    | Should   | L          | 0    | None |
| F-032 | Data Privacy/Retention         | Must     | L          | 4    | F-001, F-022 |
| F-033 | Rate Limiting/Abuse Prevention | Must     | M          | 2    | F-001, F-003, F-015 |
| F-034 | Editor Dashboard Analytics     | Could    | M          | 6    | F-010, F-011, F-029 |
| F-035 | Story Sharing                  | Should   | S          | 5    | F-014 |
| F-036 | Location-Based Feed            | Should   | S          | 5    | F-013 |
| F-037 | Admin Moderation Panel         | Must     | L          | 6    | F-022, F-023, F-026 |
| F-038 | Multilingual AI Processing     | Must     | XL         | 2    | F-005 |

## Priority Summary

| Priority | Count | Features |
|----------|-------|----------|
| Must     | 22    | F-001, F-002, F-003, F-005, F-006, F-009, F-010, F-011, F-012, F-013, F-014, F-015, F-017, F-019, F-021, F-022, F-027, F-029, F-030, F-032, F-033, F-037, F-038 |
| Should   | 14    | F-004, F-007, F-008, F-016, F-018, F-023, F-024, F-025, F-026, F-028, F-031, F-035, F-036 |
| Could    | 2     | F-020, F-034 |
| Won't    | 0     | -- |

---

# Appendix B: Unmapped PRD Ambiguities Requiring Stakeholder Clarification

1. **Story merging** -- PRD 7.4 mentions "Future: story merging" for duplicates. No timeline or scope defined. Needs product decision: is this V1 or V2?
2. **Creator edit after submission** -- Can a creator edit their submission while it is "In Progress" or "Under Review"? PRD is silent.
3. **Editor assignment model** -- Are editors assigned by city? By language? By random queue distribution? Not defined.
4. **Confirmation geographic constraint** -- Should confirmations be limited to users in the same city as the story? PRD does not specify.
5. **Reputation point values** -- What actions earn how many points? PRD gives milestones (100, 1000) but not earning rates.
6. **"HT Gifts" definition** -- What are HT Gifts? Physical merchandise? Digital rewards? Subscription upgrades? Not defined.
7. **Creator anonymity option** -- PRD states "Real name (default)" but does not clarify if anonymous or pseudonymous reporting is possible.
8. **Maximum submission frequency per level** -- Different rate limits for Basic vs. Approved vs. Trusted? Not specified.
9. **Editor SLA** -- No target time for editorial review (e.g., 2 hours, 24 hours). Needed for operational planning.
10. **Web vs. Mobile** -- PRD states "HT Mobile Application" but does not address whether a web version is planned.

---

**END OF PRD ANALYSIS v2.0**
**Status:** PRD_ANALYSIS_COMPLETE
**Next:** Root_Requirement_Agent picks up for requirement decomposition
