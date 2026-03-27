# Flutter App Task List - Mai Bhi Editor

**Extracted from:** Sprint_Task_List.md v2.0
**Agent:** Task_Breaking_Agent -> Flutter_Requirement_Agent
**Date:** 2026-03-27
**Stack:** Frontend (Flutter, BLoC, Clean Architecture)
**Total Tasks:** 38 dev tasks + QA tasks

---

## Sprint 0: Foundation

| ID | Title | TRD Ref | Feature | Dependencies | Complexity | Acceptance Criteria |
|----|-------|---------|---------|-------------|------------|---------------------|
| S0-01 | Scaffold Flutter project with Clean Architecture (BLoC, layers) | -- | -- | None | M | - data/domain/presentation layers - BLoC DI configured - Runs on iOS+Android |
| S0-04 | Generate Flutter data models from OpenAPI contract | -- | -- | S0-03 | M | - All models generated via build_runner - JSON serialization tests pass |
| S0-08 | Setup Flutter theme, design tokens, and accessibility base | TRD-CROSS-001 | F-030 | S0-01 | M | - Color palette with 4.5:1 contrast - Semantics widgets in base components |

---

## Sprint 1: Authentication

| ID | Title | TRD Ref | Feature | Dependencies | Complexity | Acceptance Criteria |
|----|-------|---------|---------|-------------|------------|---------------------|
| S1-06 | Login screen UI (BLoC pattern) | TRD-AUTH-001 | F-001 | S0-01, S0-04, S1-01 | M | - Login form with validation - Error states - Navigation on success |
| S1-07 | Auth state management + secure token storage | TRD-AUTH-002 | F-001 | S1-06 | M | - flutter_secure_storage for tokens - HTTP interceptor auto-refresh - Logout clears state |
| S1-08 | Profile screen with creator badge display | TRD-AUTH-004 | F-002 | S1-07 | M | - Profile with name, avatar, level badge - Badge icon/color per level |
| S1-09 | KYC upload screen UI | TRD-AUTH-005 | F-022 | S1-08 | M | - Document upload form - Camera/gallery picker - Progress indicator |
| S1-11 | Auth UI widget tests | TRD-AUTH-001 | F-001 | S1-07 | S | - Form validation tested - Error states tested - Navigation tested |

---

## Sprint 2: Submission

| ID | Title | TRD Ref | Feature | Dependencies | Complexity | Acceptance Criteria |
|----|-------|---------|---------|-------------|------------|---------------------|
| S2-05 | News submission form UI | TRD-SUB-002 | F-003 | S0-04, S1-07, S2-01 | L | - All mandatory fields - Inline validation - City searchable dropdown - Form state preserved on background |
| S2-06 | Multi-image picker and upload UI | TRD-SUB-003 | F-004 | S2-05 | M | - "Add More Images" gallery/camera picker - Thumbnails with reorder/delete - Client-side validation |
| S2-07 | "Posted by You" dashboard UI | TRD-SUB-006 | F-019 | S2-05 | M | - Submissions list sorted by recency - Status filter chips - Rejection reasons visible - Empty state with CTA |
| S2-08 | Offline draft support (local persistence) | TRD-SUB-007 | F-028 | S2-05, S2-07 | M | - Auto-save to Hive/SQLite - Drafts in dashboard under "Draft" - Restore all fields on tap |
| S2-10 | Submission form widget tests | TRD-SUB-002 | F-003 | S2-05 | S | - Validation tested - City dropdown tested - Background preservation tested |

---

## Sprint 3: AI Preview

| ID | Title | TRD Ref | Feature | Dependencies | Complexity | Acceptance Criteria |
|----|-------|---------|---------|-------------|------------|---------------------|
| S3-10 | "Your post corrected by AI" preview screen | TRD-AI-002 | F-005 | S0-04, S2-05, S3-02 | M | - Side-by-side original vs AI text - "Your post -- corrected by AI" header - Proceed button, no edit on AI text |
| S3-11 | AI tag suggestion UI (selectable chips) | TRD-AI-005 | F-007 | S3-10 | S | - Tag chips with deselect - Category dropdown with AI pre-selection - City unchanged |

---

## Sprint 4: Editor UI

| ID | Title | TRD Ref | Feature | Dependencies | Complexity | Acceptance Criteria |
|----|-------|---------|---------|-------------|------------|---------------------|
| S4-05 | Editor review queue UI | TRD-ED-002 | F-010 | S0-04, S4-01 | L | - Queue with cards, city filter, sort toggle - Auto-refresh 30s - Infinite scroll |
| S4-06 | Editor review detail + action UI | TRD-ED-003 | F-011 | S4-05 | L | - AI draft + original + scores shown - Approve/Edit/Reject/Correct buttons - Text input for reject/correct |
| S4-07 | False news flag UI | TRD-ED-004 | F-012 | S4-06 | S | - "Flag as False" button - Confirmation dialog - Success/error feedback |
| S4-08 | Appeal submission UI | TRD-ED-005 | F-023 | S2-07, S4-04 | M | - Appeal button on rejected submissions - Justification text area - Disabled after appeal filed |
| S4-10 | Editor UI widget tests | TRD-ED-002, 003 | F-010, F-011 | S4-06 | S | - Queue rendering tested - Action buttons tested |

---

## Sprint 5: Feed and Publishing

| ID | Title | TRD Ref | Feature | Dependencies | Complexity | Acceptance Criteria |
|----|-------|---------|---------|-------------|------------|---------------------|
| S5-06 | "News By You" feed screen UI | TRD-FEED-002 | F-013 | S0-04, S5-01 | L | - Top-level section - Story cards list - City filter, infinite scroll, pull-to-refresh - Empty state |
| S5-07 | Story card widget | TRD-FEED-003 | F-013 | S5-06 | M | - Title, image, creator+badge, counts, confirm button, city, time |
| S5-08 | Story detail page UI | TRD-FEED-004 | F-014 | S5-07 | M | - Full content, gallery, badges, like/confirm/share buttons |
| S5-09 | Search UI within "News By You" | TRD-FEED-006 | F-025 | S5-06 | M | - Search icon, input, results in card format, empty state |
| S5-10 | City preference selection + location feed | TRD-FEED-007 | F-036 | S5-06 | S | - First-time city prompt - Settings city selector - Optional GPS suggestion |
| S5-11 | Story share button integration | TRD-FEED-008 | F-035 | S5-08 | S | - Native share sheet - Deep link handling |
| S5-13 | Feed and detail UI widget tests | TRD-FEED-002, 003, 004 | F-013, F-014 | S5-08 | S | - Card rendering tested - Detail layout tested - Scroll behavior tested |

---

## Sprint 6: Community

| ID | Title | TRD Ref | Feature | Dependencies | Complexity | Acceptance Criteria |
|----|-------|---------|---------|-------------|------------|---------------------|
| S6-06 | Confirm + like UI interactions | TRD-COM-001, 002 | F-015, F-016 | S5-07, S5-08 | M | - Confirm/like buttons - Optimistic update - Auth prompt for unauthenticated |
| S6-07 | Creator public profile page UI | TRD-REP-003 | F-018 | S6-04 | M | - Name, avatar, badges, published count, story list |
| S6-08 | Block and report UI | TRD-COM-003 | F-026 | S6-07 | M | - Report form with categories - Block button - Block list in settings |
| S6-09 | Reputation badge display widget | TRD-AUTH-004, TRD-REP-001 | F-002, F-017 | S6-07 | S | - Badge collection - Point total on own profile |
| S6-11 | Community UI widget tests | TRD-COM-001, 002 | F-015, F-016 | S6-06 | S | - Button states tested - Optimistic update tested |

---

## Sprint 7: Notifications and i18n

| ID | Title | TRD Ref | Feature | Dependencies | Complexity | Acceptance Criteria |
|----|-------|---------|---------|-------------|------------|---------------------|
| S7-05 | Push notification handling (Flutter) | TRD-NOTIF-001 | F-021 | S7-01 | M | - FCM SDK integration - Foreground/background handling - Tap navigation |
| S7-06 | Notification preferences settings UI | TRD-NOTIF-002 | F-021 | S7-05 | S | - Toggle per category - Syncs with backend |
| S7-07 | Onboarding walkthrough UI | TRD-CROSS-003 | F-024 | S2-05 | M | - 4-step tooltip tour - Skippable - Community guidelines acceptance - Does not repeat |
| S7-08 | i18n setup (Flutter ARB files) | TRD-CROSS-002 | F-031 | S0-01 | L | - ARB files for Hindi+English - Language switcher - Locale detection - RTL scaffold |
| S7-10 | Notification and onboarding widget tests | TRD-NOTIF-001, TRD-CROSS-003 | F-021, F-024 | S7-07 | S | - Tap navigation tested - Onboarding flow tested - i18n strings tested |

---

## Sprint 8: Platform Features

| ID | Title | TRD Ref | Feature | Dependencies | Complexity | Acceptance Criteria |
|----|-------|---------|---------|-------------|------------|---------------------|
| S8-07 | Rate limit feedback UI | TRD-PLAT-005 | F-033 | S8-01 | S | - Rate limit message on 429 - Remaining count display |
| S8-08 | Account deletion UI in settings | TRD-PLAT-004 | F-032 | S8-03 | S | - Delete button - Confirmation dialog - Redirect to login |
| S8-09 | Analytics event SDK integration (Flutter) | TRD-PLAT-003 | F-029 | S8-06 | M | - Events for all user actions - Batched sending - Offline queuing |

---

## Sprint 9: Admin and Polish

| ID | Title | TRD Ref | Feature | Dependencies | Complexity | Acceptance Criteria |
|----|-------|---------|---------|-------------|------------|---------------------|
| S9-04 | Admin moderation panel UI | TRD-PLAT-001 | F-037 | S9-01 | L | - Tab queues: reports, KYC, appeals, flagged - Action buttons with confirmations |
| S9-05 | Editor analytics dashboard UI | TRD-PLAT-002 | F-034 | S9-02 | M | - Metrics cards - City filter - Auto-refresh |
| S9-06 | Trending badge and feed integration | TRD-FEED-005 | F-020 | S9-03 | S | - Trending badge on cards - Promoted position |
| S9-07 | Accessibility audit and fixes | TRD-CROSS-001 | F-030 | S5-08, S6-07 | L | - VoiceOver/TalkBack audit - Contrast fixes - Semantics labels - Keyboard navigation |
| S9-11 | Accessibility widget tests | TRD-CROSS-001 | F-030 | S9-07 | M | - Semantics labels verified - Contrast ratios checked - Focus order logical |

---

## Total: 38 Flutter tasks (30 dev + 8 QA)

---

**END OF FLUTTER APP TASK LIST**
