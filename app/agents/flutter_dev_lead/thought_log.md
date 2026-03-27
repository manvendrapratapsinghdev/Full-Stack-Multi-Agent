# Flutter Dev Lead - Thought Log

## Agent Config
- **Role:** Implement Flutter features per App_TRD, maintain Clean Architecture
- **Toolset:** BLoC state management, Widget composition, Flutter testing
- **Architecture:** Clean Architecture (data/domain/presentation layers)

---

## Log

### 2026-03-27 | INIT
**Status:** Awaiting Sprint 0 task S0-01 (Scaffold Flutter project).
**Architecture decisions:**
- State management: BLoC pattern
- Architecture: Clean Architecture (data → domain → presentation)
- HTTP client: Dio with interceptors
- Navigation: GoRouter
- DI: GetIt + Injectable

### 2026-03-27 | S0-01 COMPLETE - Scaffold Flutter project with Clean Architecture
**What was done:**
- Ran `flutter create --org com.ht.maibhieditor` in app/ directory (Flutter 3.41.5, Dart 3.11.3)
- Restructured to Clean Architecture with 6 feature modules: auth, submission, feed, editorial, community, profile
- Each feature has data/domain/presentation layers with proper separation
- Configured pubspec.yaml with all dependencies: flutter_bloc, go_router, dio, freezed v3, get_it, flutter_secure_storage, google_fonts, etc.
- Set up GetIt DI in injection.dart with full auth feature wiring
- Built Dio ApiClient with AuthInterceptor (JWT attach, 401 refresh, token persistence via flutter_secure_storage)
- Configured GoRouter with routes for all major screens (login, feed, story detail, submit, my-submissions, editorial queue/review, profile, creator profile)
- Created MaterialApp with BLoC providers, theming, and localization in app.dart
- Added l10n ARB files for English and Hindi
- `flutter analyze` passes with 0 issues
- `flutter test` passes (2/2)

### 2026-03-27 | S0-04 COMPLETE - Generate Flutter data models from contract.yaml
**What was done:**
- Created 15 Freezed model files covering ALL contract.yaml schemas:
  - Auth: UserProfile, LoginRequest, AuthResponse
  - Submission: SubmissionCreate, Submission, SubmissionDetail, AIReviewResult
  - Editorial: EditorialQueueItem, EditorAction
  - Feed: FeedResponse, FeedItem, StoryDetail
  - Community: ConfirmationResult, LikeResult, CreatorProfile
- All models use @freezed annotation with fromJson/toJson via json_serializable
- All snake_case API fields mapped with @JsonKey annotations
- Enums (CreatorLevel, SubmissionStatus, EditorActionType) use @JsonValue for serialization
- build_runner generated 15 .freezed.dart + 15 .g.dart files (30 total)
- Used freezed v3.2.5 + freezed_annotation v3.1.0 for Dart 3.11 compatibility

**Dependency resolution notes:**
- freezed v2 incompatible with Dart 3.11 (analyzer conflicts with bloc_test)
- Upgraded to freezed v3 + freezed_annotation v3 to resolve
- Removed hive_generator (source_gen v1 conflict with freezed's source_gen v2)
- json_annotation v4.11.0 resolved (warning about ^4.9.0 constraint is benign)

### 2026-03-27 | S0-08 COMPLETE - Setup Flutter theme, design tokens, accessibility base
**What was done:**
- Created AppColors with HT brand red primary (#CC0000, 4.63:1 contrast on white — AA pass)
- Dark variant #990000 (7.0:1 contrast) for pressed/focused states
- Semantic status colors: in_progress=amber, under_review=blue, published=green, rejected=red
- Tinted chip backgrounds for each status (dark text on light bg passes 4.5:1)
- Created AppTheme with Material 3 light + dark themes
- Text theme uses Google Fonts (Noto Sans) with proper scaling hierarchy (11px–32px)
- All input decorations, buttons, cards, chips, and bottom nav styled consistently
- Semantics widgets used in LoginScreen (header, button, label annotations)
- All foreground/background combinations verified for WCAG 2.1 AA (>= 4.5:1)
- statusColor() and statusBackgroundColor() helpers for dynamic status styling

### 2026-03-27 | S1-06 COMPLETE - Login Screen UI (BLoC pattern)
**What was done:**
- Enhanced LoginScreen with full production-quality implementation:
  - HT branded logo area (red rounded container with "HT" text)
  - "Mai Bhi Editor" title + "Citizen journalism by Hindustan Times" subtitle
  - Email field with regex validation, email keyboard, autofill hints, focus management
  - Password field with obscure toggle (eye icon), min 6 chars validation
  - "Sign In" button with loading state (CircularProgressIndicator) and disabled state
  - Error display via floating SnackBar with error icon and dismiss action
  - Navigation to /feed on AuthAuthenticated state
  - Keyboard dismissal before form submission
  - AutofillGroup for password manager support
  - Focus node management (email -> password -> submit)
  - Responsive layout: ConstrainedBox(maxWidth: 400), adaptive spacing for small screens
  - Full Semantics: header, textField, button, image labels for VoiceOver/TalkBack
  - Terms of Service footer text

### 2026-03-27 | S1-07 COMPLETE - Auth state management + secure token storage
**What was done:**
- **AuthLocalDataSource** (abstract + impl): flutter_secure_storage wrapper for token CRUD
  - saveTokens(), getAccessToken(), getRefreshToken(), clearTokens(), hasToken()
- **AuthRemoteDataSource** enhanced: added logout() (POST /auth/logout, best-effort) and refreshToken() (POST /auth/refresh)
- **AuthRepositoryImpl** refactored to use both local + remote data sources:
  - login: calls remote, persists tokens in both local storage and interceptor
  - logout: best-effort server call, then clears local + interceptor tokens
  - isAuthenticated: delegates to local data source
- **AuthBloc** enhanced:
  - New event: AuthTokenRefreshed (re-fetches profile after interceptor refresh)
  - Error handling: catches ServerException/NetworkException with friendly messages
  - _friendlyMessage() maps HTTP status codes to user-facing text (401, 403, 404, 422, 429)
  - CheckAuthStatus: on 401 during profile fetch, auto-logout and emit Unauthenticated
- **GoRouter redirect logic** fully wired:
  - _publicRoutes set for /login
  - Unauthenticated + protected route -> redirect to /login
  - Authenticated + on /login -> redirect to /feed
  - AuthInitial/AuthLoading -> no redirect (wait for auth check)
- All new classes registered in injection.dart (GetIt)
- `flutter analyze` passes with 0 issues

### 2026-03-27 | S1-08 COMPLETE - Profile Screen with creator badge display
**What was done:**
- **CreatorBadgeWidget**: reusable badge component with per-level styling:
  - basic_creator: grey shield icon
  - ht_approved_creator: blue verified checkmark
  - trusted_reporter: gold star
  - Tinted background container, optional label, configurable size
- **ProfileScreen** fully implemented:
  - Avatar: CircleAvatar with NetworkImage or initials fallback (2-letter)
  - Name (header) + email
  - CreatorBadgeWidget for level display
  - Stats row: Reputation points (gold star) + Stories published (green article)
  - Details card: City, Level, Badges (if any)
  - "Upgrade to Approved Creator" button (navigates to /kyc) for basic_creator only
  - Logout button with confirmation dialog (AlertDialog)
  - Pull-to-refresh via RefreshIndicator
  - BlocConsumer: listens for AuthUnauthenticated -> redirect to /login, AuthError -> SnackBar
  - Fallback UI for unexpected states with retry button
  - Full Semantics on all interactive and informational elements
- KYC route (/kyc) added to GoRouter

### 2026-03-27 | S1-09 COMPLETE - KYC Upload Screen UI
**What was done:**
- **KYC domain layer**: KycVerificationStatus enum (none/pending/approved/rejected), KycDocumentType enum (aadhaar/pan/passport/voterId) with displayName and apiValue extensions
- **KycRepository** abstract interface: getKycStatus(), uploadDocument()
- **KycRemoteDataSource**: GET /auth/kyc/status, POST /auth/kyc/upload with FormData + MultipartFile
- **KycRepositoryImpl**: delegates to remote data source
- **KycBloc** with events/states:
  - Events: KycStatusRequested, KycDocumentTypeChanged, KycImagePicked, KycUploadRequested
  - State: single KycState with copyWith (verificationStatus, isLoadingStatus, selectedDocumentType, pickedFilePath, isUploading, uploadProgress, errorMessage, successMessage)
  - hasSubmitted computed property for pending/approved check
- **KycUploadScreen** full implementation:
  - On load: fetches KYC status from API
  - If pending/approved: shows status view with icon, title, description, refresh button
  - If none/rejected: shows upload form
  - Rejected banner warning with red styling
  - Document type selector: ChoiceChip grid for all 4 types
  - Image picker area: tap to show camera/gallery bottom sheet, preview selected image
  - Upload button with loading spinner (disabled when no image or uploading)
  - Upload progress: LinearProgressIndicator + percentage text
  - Guidelines card with 4 helpful tips
  - Error/success messages via floating SnackBars
  - Full Semantics on all elements
- All KYC classes registered in injection.dart
- `flutter analyze` passes with 0 issues

### 2026-03-27 | S2-05, S2-06, S2-07, S2-08 COMPLETE - Submission Form, Multi-Image, Dashboard, Offline Drafts

**S2-05: News Submission Form UI**
- **Domain layer**: CreateSubmissionUseCase, GetMySubmissionsUseCase, GetSubmissionDetailUseCase in domain/usecases/
- **Data layer**: SubmissionRemoteDataSource (POST /submissions, GET /submissions/my, GET /submissions/{id}, POST /submissions/{id}/images), SubmissionRepositoryImpl with draft merge logic
- **SubmissionFormBloc** with events: TitleChanged, DescriptionChanged, CityChanged, CoverImagePicked, CoverImageRemoved, AdditionalImageAdded, AdditionalImageRemoved, ImagesReordered, FormSubmitted, DraftLoaded, DraftSaveRequested, DraftDeleteRequested, FormReset
- **SubmissionFormState**: all form fields, inline validation errors per field, isSubmitting, isSuccess, isDraftSaved, draftId
- **SubmissionFormScreen**: Title (max 200, char counter), Description (multiline max 5000, char counter), City searchable autocomplete (30 major Indian cities), Cover image picker (camera/gallery via bottom sheet) with preview, Additional images with reorder/delete, Submit with loading indicator, inline validation, success -> navigate to /my-submissions
- Added `draft` status to SubmissionStatus enum

**S2-06: Multi-Image Picker and Upload UI**
- "Add More Images" button opens camera/gallery bottom sheet
- Selected images displayed as list with thumbnails
- Each thumbnail has drag handle for reorder (ReorderableListView) and delete (X) button
- Client-side validation: max 10MB per image, extension check (JPG/PNG/WebP)
- Upload via SubmissionRemoteDataSource.uploadAdditionalImages()

**S2-07: "Posted by You" Dashboard UI**
- **MySubmissionsBloc**: events: LoadMySubmissions, FilterByStatus, LoadMore, Refresh, DeleteDraft; states: submissions list, isLoading, isLoadingMore, hasMore, filterStatus, errorMessage
- **MySubmissionsScreen**: Status filter chips (All, Draft, In Progress, Under Review, Published, Rejected) with semantic colors from AppColors, Submission cards with title/thumbnail/status badge/city/relative time, Empty state with illustration + CTA, Error state with retry, Pull-to-refresh (RefreshIndicator), Infinite scroll pagination, Tap -> detail or edit draft, FAB "+" for new submission
- Swipe-to-dismiss for drafts with confirmation dialog

**S2-08: Offline Draft Support**
- **DraftLocalDataSource** (abstract + impl): SharedPreferences-based persistence for DraftSubmission objects
- DraftSubmission model with toJson/fromJson, toSubmission() converter
- Auto-save via "Save Draft" button in AppBar + "Save as draft?" dialog on back navigation (PopScope)
- Drafts appear in dashboard with "Draft" status chip (grey color)
- Tap draft -> navigates to /submit?draftId=xxx, form fields restored
- Delete draft via swipe-to-dismiss or DraftDeleteRequested event
- SubmissionRepositoryImpl merges local drafts with remote submissions in getMySubmissions()

**Infrastructure updates:**
- Added `draft` to SubmissionStatus enum + AppColors (statusDraft, statusDraftBg)
- Added submissionImages() endpoint to ApiConstants
- Registered all submission dependencies in injection.dart (datasources, repo, usecases, BLoCs)
- GoRouter /submit route updated to accept ?draftId query parameter
- `flutter analyze` passes with 0 issues

### 2026-03-27 | S3-10, S3-11 COMPLETE - AI Preview Screen + Tag Suggestion UI

**S3-10: "Your post corrected by AI" preview screen**

*Data/Domain layer:*
- Added `triggerAiProcessing(submissionId)` to `SubmissionRemoteDataSource` (abstract + impl) — calls POST /submissions/{id}/process-ai
- Added `processAi(id)` endpoint to `ApiConstants`
- Wired `triggerAiReview()` in `SubmissionRepositoryImpl` (previously threw UnimplementedError)
- Created `TriggerAiProcessingUseCase` in domain/usecases/
- Registered `GetSubmissionDetailUseCase` and `TriggerAiProcessingUseCase` in injection.dart

*Presentation - AiPreviewBloc:*
- Events: `LoadAiPreview(submissionId)`, `TriggerAiProcessing(submissionId)`, `AcceptAndProceed`, `ToggleTag(tag)`, `UpdateCategory(category)`
- States: `AiPreviewInitial`, `AiPreviewLoading`, `AiPreviewProcessing(submissionDetail)`, `AiPreviewLoaded(submissionDetail, aiReview, selectedTags, selectedCategory)`, `AiPreviewError(message, submissionId?)`
- Flow: LoadAiPreview fetches detail; if aiReview present -> Loaded; if null -> Processing -> triggers AI -> Loaded
- Full error handling for ServerException, NetworkException, generic
- Registered `AiPreviewBloc` factory in injection.dart

*UI - AiPreviewScreen:*
- Header: "Your post -- corrected by AI" (em dash)
- Two-panel layout: Original Text (grey background) + AI Corrected Text (green background)
- Word-level diff highlighting: new/changed words shown with green background + bold
- If text identical: "No changes needed! Your text looks great." banner with green check
- Safety score indicator: green check (> 0.7), yellow warning (0.4-0.7), red alert (< 0.4) — with percentage
- Confidence score badge (blue)
- Detected language badge (amber)
- Safety flags banner: if hate speech, toxicity, or spam detected -> red warning banner with chips + description
- "Proceed to Review" button with send icon in bottom action bar with shadow
- Processing state: pulsing auto_fix_high icon + linear progress indicator + descriptive text
- Loading state: circular progress with "Loading submission..." text
- Error state: retry button (reloads) + go back button
- All elements have Semantics annotations (header, label, liveRegion, button)
- SelectableText used for text panels so content can be copied
- Route: `/submissions/:id/ai-preview` registered in GoRouter as `aiPreview`

**S3-11: AI tag suggestion UI (selectable chips)**
- `_TagSuggestionSection`: FilterChip widgets for each suggested tag, pre-selected by default
- Users can tap to deselect/reselect; selected state managed via `ToggleTag` event in BLoC
- Styled consistently with existing app filter chips (blue accent when selected)
- `_CategorySection`: DropdownButtonFormField with 15 categories, AI-suggested one marked with "AI" badge
- User can override category; managed via `UpdateCategory` event in BLoC
- `_CityReadOnly`: TextFormField with readOnly=true, enabled=false, "Read-only" label badge
- City displayed but NOT editable (as per PRD)
- Tags and category selections stored in `AiPreviewLoaded` state for submission on proceed

**Infrastructure updates:**
- `ApiConstants.processAi(id)` added for POST /submissions/{id}/process-ai
- GoRouter: added `/submissions/:id/ai-preview` route
- injection.dart: registered GetSubmissionDetailUseCase, TriggerAiProcessingUseCase, AiPreviewBloc
- `flutter analyze` passes with 0 issues

### 2026-03-27 | S4-05, S4-06, S4-07, S4-08 COMPLETE - Editor Queue, Review Detail, False News Flag, Appeal UI

**S4-05: Editor review queue UI**

*Data layer:*
- `EditorialRemoteDataSource` (abstract + impl): GET /editorial/queue (with city, sortBy, page params), GET /submissions/{id} (detail), POST /editorial/{id}/action, POST /editorial/{id}/flag-false, GET /editorial/appeals, POST /editorial/appeals/{id}/review, POST /submissions/{submissionId}/appeal, GET /submissions/{submissionId}/appeal
- `EditorialRepositoryImpl`: delegates all calls to remote data source

*Domain layer:*
- `EditorialRepository` (abstract): expanded with flagFalseNews(), getAppeals(), reviewAppeal(), submitAppeal(), getAppealForSubmission(), getSubmissionDetail()
- `GetEditorialQueueUseCase`: fetches queue with city/sortBy/page params
- `PerformEditorActionUseCase`: performs approve/edit/reject/mark_correction actions
- `FlagFalseNewsUseCase`: flags a published story as false
- `SubmitAppealUseCase`: submits appeal for rejected submission
- `GetAppealUseCase`: fetches appeal status for a submission
- `Appeal` entity: id, submissionId, justification, status (pending/approved/rejected), reviewNotes, timestamps

*Presentation - EditorialQueueBloc:*
- Events: LoadQueue, FilterByCity, ChangeSortOrder, LoadMore, RefreshQueue, AutoRefreshQueue
- State: items list, isLoading, isLoadingMore, hasMore, cityFilter, sortOrder (confidenceScore/recent/mostConfirmed), currentPage, errorMessage, availableCities
- Auto-refresh via 30-second Timer.periodic (silent, no loading indicators)
- City list extracted from queue items for dropdown

*UI - EditorialQueueScreen:*
- City filter dropdown (All Cities + dynamic list)
- Sort toggle dropdown (Confidence Score | Recent | Most Confirmed)
- Queue list cards with: title + thumbnail, creator name + badge level, AI confidence score bar (green/yellow/red), safety score indicator, community confirmations count, duplicate alert badge, time since submission
- Tap card -> navigates to /editorial/{id}
- Pull-to-refresh, infinite scroll, auto-refresh every 30s
- Empty state: "No submissions pending review"
- Error state with retry button

**S4-06: Editor review detail + action UI**

*Presentation - EditorReviewBloc:*
- Events: LoadReview(submissionId), PerformAction(action), UpdateEditedText, FlagAsFalse
- States: EditorReviewInitial, EditorReviewLoading, EditorReviewLoaded (with editedTitle/Description, isFlaggedFalse, isFlagging), EditorReviewSubmitting, EditorReviewActionSuccess, EditorReviewFlagSuccess, EditorReviewError

*UI - EditorReviewScreen:*
- Top: submission metadata card (creator, city, time, status badge)
- Two-panel comparison: "Original Text" (grey) + "AI Corrected Text" (blue)
- Safety score card with progress bar + individual flags (hate speech, toxicity, spam) with DETECTED/Clear indicators
- Duplicate alert card with warning styling
- Community confirmations info row
- Creator reputation card (name, reputation points, stories published)
- Action bar with 4 buttons: Approve (green), Edit (blue), Reject (red), Correct (orange)
  - Approve: confirmation dialog -> publishes
  - Edit: dialog with editable title + description fields
  - Reject: dialog with required rejection reason text input
  - Mark Correction: dialog with required correction notes text input
- Loading state during action submission
- Success: SnackBar + navigate back to queue

**S4-07: False news flag UI**
- For published stories: bottom action bar shows "Flag as False News" button (red)
- Confirmation dialog: "Are you sure? This will remove the story from the feed and deduct creator reputation."
- On confirm: calls flagFalseNews API via BLoC
- Success: SnackBar + navigate back to queue
- Disabled if already flagged (button text changes to "Already Flagged")

**S4-08: Appeal submission UI**
- On MySubmissionsScreen: rejected submission cards show "Appeal" button (red tint)
- Tapping opens AppealBottomSheet with:
  - Handle bar + "Appeal Rejection" title
  - Rejection reason displayed (read-only, red tinted container)
  - Justification text area (required, min 50 chars with validation)
  - Submit appeal button
- AppealBloc: events LoadAppealStatus, SubmitAppealEvent; states AppealInitial, AppealLoading, AppealNotFiled, AppealPending, AppealReviewed, AppealSubmitting, AppealSubmitted, AppealError
- After appeal submitted: shows "Appeal Pending" view with hourglass icon (disabled submit)
- If appeal reviewed: shows "Appeal Approved" or "Appeal Rejected" status with appropriate icon/color

**Infrastructure updates:**
- ApiConstants: added flagFalseNews(), editorialAppeals, reviewAppeal(), submitAppeal() endpoints
- AppColors: added confidenceHigh/Medium/Low colors + backgrounds, confidenceColor()/confidenceBackgroundColor() helpers, actionApprove/Edit/Reject/Correction colors, safetyGood/Warning/Danger colors
- injection.dart: registered EditorialRemoteDataSource, EditorialRepository, all 5 use cases, EditorialQueueBloc, EditorReviewBloc, AppealBloc
- GoRouter routes already existed from S0-01
- `flutter analyze` passes with 0 issues

### 2026-03-27 | S5-06, S5-07, S5-08, S5-09, S5-10, S5-11 COMPLETE - Feed, Story Card, Detail, Search, City Pref, Share

**S5-06: "News By You" feed screen UI**

*Data layer:*
- `FeedRemoteDataSource` (abstract + impl): GET /feed (with city, cursor, limit), GET /feed/{id}, GET /feed/search, GET /feed/{id}/share
- `FeedRepositoryImpl`: delegates all calls to remote data source

*Domain layer:*
- `FeedRepository` (abstract): updated with searchStories() and getShareableLink() methods
- `GetFeedUseCase`: fetch feed with optional city filter and cursor pagination
- `GetStoryDetailUseCase`: fetch full story detail by ID
- `SearchStoriesUseCase`: search stories by query with cursor pagination
- `GetShareableLinkUseCase`: get shareable deep-link URL for a story

*Presentation - FeedBloc:*
- Events: LoadFeed, LoadMoreFeed, FilterFeedByCity, RefreshFeed, SearchStories, ClearSearch
- State: stories list, isLoading, isLoadingMore, hasMore, cityFilter, nextCursor, errorMessage, isSearching, searchQuery, searchResults, isSearchLoading
- Search with debounce (300ms) via Timer + Completer pattern
- Cursor-based infinite scroll pagination (20 items per page)

*UI - FeedScreen:*
- App bar with "News By You" title (shows city subtitle when filtered)
- Search icon toggles inline search bar with debounced text input
- City filter icon with badge indicator (shows when city is active)
- Story cards in a ListView with pull-to-refresh (RefreshIndicator)
- Infinite scroll (LoadMoreFeed on scroll within 200px of bottom)
- Empty state per city: "No stories yet from [City]. Be the first reporter!"
- Error state with retry button
- FAB: "+" navigates to /submit
- First-time city prompt check on init

**S5-07: Story card widget**
- `StoryCard`: reusable across feed, search results, and profile screens
- Cover image via cached_network_image with placeholder (article icon) and error (broken image icon)
- Title (max 2 lines, ellipsis, titleMedium weight 600)
- Creator row: person icon + name + CreatorBadgeWidget (parsed from string level)
- Like count (heart icon, red) + confirmation count (check icon, green)
- Optional "Confirm" button: outlined when not confirmed, filled green when confirmed
- City tag chip with location pin icon
- Relative time display ("Just now", "2m ago", "3h ago", "Yesterday", "5d ago", "5 Mar")
- Tap entire card navigates to /feed/{id} via go_router
- Count formatting: K for thousands, M for millions

**S5-08: Story detail page UI**

*Presentation - StoryDetailBloc:*
- Events: LoadStoryDetail(id), ToggleLike, ToggleConfirm, ShareStory
- State: story detail, isLoading, errorMessage, isLiked, isConfirmed, likesCount, confirmationsCount, isSharing
- Optimistic like/confirm toggles with rollback on API failure
- Uses CommunityRepository for like/confirm API calls

*UI - StoryDetailScreen:*
- SliverAppBar with hero cover image (260px expanded, pinned)
- Share button in app bar
- Title (headlineMedium, bold) + relative time
- Creator row: CircleAvatar (CachedNetworkImage or person fallback), name, CreatorBadgeWidget, city, "View Profile" TextButton
- Verification badges: "AI Verified" chip (blue), "Editor Verified" chip (green)
- Full description text, horizontal image gallery, editor verifier row
- Stats row: likes, confirmations, "Confirmed by N users"
- Action bar: Like (toggle, red), Confirm (toggle, green), Share (outlined, grey)

**S5-09: Search UI within "News By You"**
- Inline search bar with debounced text input (300ms)
- Results in StoryCard format, empty state "No stories found for '[query]'"
- Recent searches stored locally (SharedPreferences, last 5), clear all button

**S5-10: City preference selection + location feed**
- `CityPreferenceDataSource`: SharedPreferences storage for preferred city, prompt shown flag, recent searches
- `CityPromptSheet`: first-time bottom sheet with 17 cities + skip option
- `CityFilterSheet`: bottom sheet with "All Cities" + city list + selected indicator
- City filter saved to preferences for persistence across sessions

**S5-11: Story share button integration**
- `ShareButton` widget: icon button (app bar) and action button (with label) modes
- Fetches shareable link via GetShareableLinkUseCase
- Composes share text with title + URL
- Deep link: /stories/:id route redirects to /feed/:id in GoRouter

**Community data layer (prerequisite for StoryDetailBloc):**
- `CommunityRemoteDataSource` + `CommunityRepositoryImpl` registered in injection.dart

**Infrastructure:**
- `ApiConstants`: added feedSearch, storyShare(id)
- `AppRouter`: BlocProvider wrappers for Feed + StoryDetail, deep link route
- `injection.dart`: all feed + community dependencies registered
- `core/utils/relative_time.dart`: formatRelativeTime() helper
- `flutter analyze` passes with 0 issues

### 2026-03-27 | S6-06, S6-07, S6-08, S6-09 COMPLETE - Confirm/Like UI, Creator Profile, Block/Report, Badge Collection

**S6-06: Confirm + like UI interactions**

*Domain layer:*
- `ConfirmStoryUseCase`: delegates to CommunityRepository.confirmStory()
- `LikeStoryUseCase`: delegates to CommunityRepository.likeStory()
- Both were already present on CommunityRepository abstract; use cases wrap them for Clean Architecture

*StoryCard widget updates:*
- Added interactive `onLike` callback + `isLiked` state
- Added `likesOverride` and `confirmationsOverride` for parent-driven optimistic values
- Converted to StatefulWidget with TickerProviderStateMixin for animations
- Like button: heart icon toggles between outlined (favorite_border) and filled (favorite) with red color
- Confirm button: outlined -> filled green on toggle
- Scale animation (TweenSequence 1.0->1.3->1.0) on both like and confirm tap
- AnimatedSwitcher for smooth icon swap on like toggle
- Auth check: if user not authenticated, shows "Sign In Required" dialog with Cancel/Sign In buttons
- Optimistic UI: instantly toggles local state + counts, parent callback triggers API call

*StoryDetailScreen updates:*
- Action bar buttons now have animated scale transitions on state change (AnimatedController + ScaleTransition)
- Like and confirm detect state changes via didUpdateWidget and trigger bounce animation
- Counter animation: TweenAnimationBuilder<int> for likes/confirmations counts in stats row
- Auth check on like/confirm tap with login prompt dialog
- Added "Report Story" option in app bar popup menu

**S6-07: Creator public profile page UI**

*Domain layer:*
- `GetCreatorProfileUseCase`: fetches CreatorProfile by ID from CommunityRepository

*Presentation - CreatorProfileBloc:*
- Events: LoadCreatorProfile(id), LoadCreatorStories(id), BlockCreator(id), UnblockCreator(id), ReportCreator(id, reason, details)
- States: profile, stories list (FeedItem), isLoading, isLoadingStories, errorMessage, isBlocked, reportSubmitted
- Auto-triggers LoadCreatorStories after profile loads
- Stories fetched via FeedRepository.searchStories() (using creator name)
- Full error handling for ServerException/NetworkException

*UI - CreatorProfileScreen:*
- SliverAppBar with creator name + 3-dot popup menu (Block/Unblock, Report)
- Profile header: large CircleAvatar (52px radius) with CachedNetworkImage or initials fallback, name, CreatorBadgeWidget
- Stats row: 3 cards (Reputation points, Stories published, Accuracy rate %)
- Badge collection: horizontal scrollable BadgeCollectionWidget
- Join date: calendar icon + "Joined Month Year" formatted with intl DateFormat
- Published Stories section: SliverList of StoryCard widgets (reused from feed)
- Empty states for loading, error (retry), no stories
- BlocConsumer: listens for errors (SnackBar) and reportSubmitted confirmation
- Route: /creators/:id with BlocProvider<CreatorProfileBloc> wrapper in GoRouter

**S6-08: Block and report UI**

*Data layer additions:*
- `CommunityRemoteDataSource`: added blockCreator(id) -> POST /users/{id}/block, report() -> POST /reports, getBlockedCreators() -> GET /users/blocked
- `CommunityRepository` interface: added blockCreator(), report(), getBlockedCreators(), getCreatorStoryIds()
- `CommunityRepositoryImpl`: delegates all new methods to remote data source
- `ApiConstants`: added blockCreator(id), reports, blockedCreators endpoints

*Domain use cases:*
- `BlockCreatorUseCase`: toggle block for a creator, returns bool
- `ReportUseCase`: submit report with targetType, targetId, reason, optional details
- `GetBlockedCreatorsUseCase`: fetch list of blocked creators

*Block Creator UI:*
- "Block Creator" / "Unblock Creator" in creator profile popup menu (dynamic based on isBlocked state)
- Confirmation dialog: "Block [name]? Their stories will no longer appear in your feed."
- Unblock dialog: "Unblock [name]? Their stories will appear in your feed again."
- Block/unblock toggles isBlocked state in CreatorProfileBloc

*Report UI (ReportBottomSheet):*
- Reusable bottom sheet for both story and creator reporting
- RadioGroup with 5 reason categories: Misinformation, Hate Speech, Spam, Harassment, Other
- Optional details TextField (max 500 chars, 3 lines)
- Submit button (disabled until reason selected, shows loading spinner)
- Success view: checkmark icon + "Report submitted. Our team will review it."
- Used from StoryDetailScreen (popup menu -> "Report Story") and CreatorProfileScreen
- Direct CommunityRepository call from widget (no separate BLoC needed)

*Blocked Creators Screen:*
- Route: /blocked-creators accessible from profile screen "Blocked Creators" button
- Shows list of blocked CreatorProfile entries with CircleAvatar, name, stories count
- "Unblock" button per creator with confirmation dialog
- Optimistic removal from list on unblock + SnackBar
- Empty state: "No blocked creators."
- Error state with retry, pull-to-refresh

**S6-09: Reputation badge display widget (BadgeCollectionWidget)**

*BadgeCollectionWidget:*
- Takes `List<String> badges` + optional `vertical` layout flag
- Badge type mapping with distinct icon, color, and background:
  - "Century Reporter" (100pts): bronze star icon on warm bg
  - "Community Voice" (500pts): silver megaphone (campaign) icon on grey bg
  - "HT Gifts Eligible" (1000pts): gold gift icon on amber bg
  - "Trusted Reporter": purple diamond shield icon on lavender bg
  - "HT Approved": blue verified checkmark icon on blue bg
  - Unknown badges: grey military_tech icon with generic styling
- Horizontal scroll mode: 90px-height ListView.separated with 140px-wide badge tiles
- Vertical mode: Column layout with SizedBox spacers
- Each badge tile: circular icon container + name text (labelMedium, bold, colored)
- Semantics on each tile: "Badge: [name]. [description]"
- Empty state: military_tech outline icon + "No badges earned yet. Start reporting to earn badges!" text
- Integrated into ProfileScreen after details card (replaces inline badges text)
- Integrated into CreatorProfileScreen in "Badges" section
- ProfileScreen also gained "Blocked Creators" button linking to /blocked-creators

**Infrastructure updates:**
- `ApiConstants`: added blockCreator(id), reports, blockedCreators
- `CommunityRepository`: expanded with blockCreator(), report(), getBlockedCreators(), getCreatorStoryIds()
- `injection.dart`: registered ConfirmStoryUseCase, LikeStoryUseCase, GetCreatorProfileUseCase, BlockCreatorUseCase, ReportUseCase, GetBlockedCreatorsUseCase, CreatorProfileBloc
- `AppRouter`: /creators/:id wrapped with BlocProvider<CreatorProfileBloc>, added /blocked-creators route + AppRoutes.blockedCreators constant
- `flutter analyze` passes with 0 issues

### 2026-03-27 | S7-05, S7-06, S7-07, S7-08 COMPLETE - Notifications, Preferences, Onboarding, i18n

**S7-05: Push notification handling (Flutter)**

*Domain layer:*
- `NotificationEntity`: entity with id, title, body, category (enum: storyApproved, storyRejected, correctionNeeded, milestone, badge, trending, communityConfirmation, general), isRead, createdAt, targetId, targetType
- `NotificationPreferences`: entity with storyUpdates, milestonesBadges, trendingStories, communityActivity toggles + fromJson/toJson
- `NotificationRepository` (abstract): getNotifications, markAsRead, markAllAsRead, registerDevice, getPreferences, updatePreferences
- `GetNotificationsUseCase`: paginated notification fetch
- `RegisterDeviceUseCase`: device token registration

*Data layer:*
- `NotificationRemoteDataSource` (abstract + impl): GET /notifications, POST /notifications/{id}/read, POST /notifications/read-all, POST /devices/register, GET /notifications/preferences, PUT /notifications/preferences
- `NotificationRepositoryImpl`: delegates to remote data source

*Presentation - NotificationBloc:*
- Events: LoadNotifications, MarkNotificationAsRead, MarkAllNotificationsAsRead, LoadMoreNotifications, RegisterDeviceToken
- State: notifications list, isLoading, isLoadingMore, unreadCount, hasMore, currentPage, errorMessage
- Infinite scroll pagination (20 per page), unread count tracking
- Best-effort mark-as-read (no UX disruption on failure)

*FCM Setup (mock-ready):*
- `PushNotificationService` abstract: initialize, getToken, requestPermission, setOnNotificationTap, setOnForegroundNotification, dispose
- `MockFcmService`: logs all operations via dart:developer, returns mock token, simulates foreground notifications and taps for testing
- `NotificationRouter`: static routeFromData() maps target_type (story, submission, profile, editorial) to app routes
- Initialized in main.dart before runApp
- Registered as lazy singleton in injection.dart

*UI - NotificationsScreen:*
- List with NotificationTile items: category icon (green check for approved, red X for rejected, gold star for milestone, fire for trending, etc.), title + body text, relative time, unread indicator (blue dot)
- Tap: marks as read + navigates to target screen via NotificationRouter
- "Mark all read" button in app bar (shows when unread count > 0)
- Pull-to-refresh, infinite scroll pagination
- Empty state: "No notifications yet" with description
- Error state with retry button
- NotificationBellWidget: badge with unread count on notification bell icon, placed in feed screen app bar

**S7-06: Notification preferences settings UI**

*Presentation - SettingsBloc:*
- Events: LoadSettings, ToggleNotificationPreference(category, value), ChangeLocale(locale)
- State: isLoading, NotificationPreferences, locale, errorMessage
- Optimistic toggle update with revert on server failure
- Locale persisted to SharedPreferences

*UI - SettingsScreen:*
- Section: "Notification Preferences" with 4 SwitchListTile toggles:
  - Story Updates (approved/rejected/corrections)
  - Milestones & Badges
  - Trending Stories in My City
  - Community Activity (confirmations on my stories)
- Each toggle: icon, title, subtitle, instant save to backend
- Section: "Language" with English/Hindi selection (check_circle indicator)
- App version footer
- Accessible from profile screen via new "Settings" OutlinedButton
- Route: /settings in GoRouter with BlocProvider wrapper

**S7-07: Onboarding walkthrough UI**

*Data layer:*
- `OnboardingPreferences`: SharedPreferences wrapper for onboarding_complete and guidelines_accepted flags
- isOnboardingComplete(), markOnboardingComplete(), resetOnboarding()
- Registered as lazy singleton in injection.dart

*UI - OnboardingScreen:*
- 4-step PageView walkthrough:
  1. "Welcome to News By You" (newspaper icon, red): citizen reporting explanation
  2. "AI Assists Your Writing" (auto_awesome icon, blue): AI rephrase explanation
  3. "Editors Verify" (verified_user icon, green): editorial review explanation
  4. "Earn Reputation" (emoji_events icon, gold): points/badges/levels explanation
- Each page: OnboardingPage widget with colored circle icon area (160px), title, description
- DotIndicator: animated dots with active indicator (24px wide, red) vs inactive (8px, grey)
- "Skip" button on non-last pages (jumps to last page)
- "Next" button on pages 1-3
- Last page: CheckboxListTile for community guidelines acceptance + "Accept & Continue" button (disabled until accepted)
- After completion: marks onboarding_complete in SharedPreferences, navigates to /feed
- Route: /onboarding in GoRouter, added to _publicRoutes (no auth required)

**S7-08: i18n Flutter setup**

*ARB expansion:*
- English (app_en.arb): expanded from 37 to 100+ translated strings
- Hindi (app_hi.arb): expanded from 37 to 100+ translated strings
- Categories covered:
  - Navigation labels (feed, profile, notifications, settings, submissions, editorial, etc.)
  - Screen titles (myProfile, editorialQueue, notifications, settings, etc.)
  - Button labels (signIn, submit, retry, cancel, discard, saveDraft, next, skip, acceptAndContinue, etc.)
  - Status labels (draft, inProgress, underReview, published, rejected)
  - Error messages (networkError, serverError, unexpectedError, rateLimitError, accountSuspended, etc.)
  - Empty state messages (noStoriesYet, noNotificationsYet, noResults, etc.)
  - Notification messages (storyApproved, storyRejected, milestone, trending, etc.)
  - Onboarding content (welcome, AI, editors, reputation titles + descriptions)
  - Parameterized strings (confirmations with count, likes with count, noStoriesInCity with city, etc.)

*Flutter localization setup:*
- l10n.yaml: arb-dir: lib/l10n, template: app_en.arb, output: app_localizations.dart
- AppLocalizations.delegate added to MaterialApp.router localizationsDelegates
- AppLocalizations.supportedLocales used for supportedLocales
- `flutter gen-l10n` generates app_localizations.dart, app_localizations_en.dart, app_localizations_hi.dart

*Language switcher:*
- SettingsBloc manages locale state, persists to SharedPreferences
- Global SettingsBloc provider in app.dart wraps MaterialApp.router
- BlocBuilder<SettingsBloc> rebuilds MaterialApp with new locale on language change
- Language selector in SettingsScreen with English/Hindi options

**Infrastructure updates:**
- `ApiConstants`: added notifications, notificationRead(id), notificationsReadAll, devicesRegister, notificationPreferences endpoints
- `AppRouter`: added /notifications, /settings, /onboarding routes + AppRoutes constants; /onboarding added to _publicRoutes
- `injection.dart`: registered MockFcmService (as PushNotificationService), NotificationRemoteDataSource, NotificationRepository, GetNotificationsUseCase, RegisterDeviceUseCase, NotificationBloc, SettingsBloc, OnboardingPreferences
- `app.dart`: updated to use AppLocalizations delegates, added global NotificationBloc + SettingsBloc providers, BlocBuilder for locale-driven MaterialApp rebuild
- `main.dart`: initializes PushNotificationService before runApp
- Profile screen: added "Settings" button navigating to /settings
- Feed screen: added NotificationBellWidget in app bar with unread badge + navigation to /notifications
- `flutter analyze` passes with 0 issues

### 2026-03-27 | S8-08, S8-09, S9-04, S9-05, S9-06, S9-07 COMPLETE - Data Privacy, Analytics, Admin Panel, Editor Analytics, Trending, Accessibility

**S8-08: Account deletion UI in settings**

*Settings screen updates:*
- Added "Data & Privacy" section with:
  - Consent status display (green check, "granted")
  - "Withdraw Consent" option (triggers account deletion flow)
  - "Download My Data" button (placeholder with "coming soon" SnackBar)
  - "About Your Data" info card explaining data collection (name, email, stories, interactions, device info, usage analytics)
- Added "Danger Zone" section with "Delete Account" button (red outlined)
- Delete Account dialog:
  - Warning text: "Your account will be deleted after 30 days. Log in during this period to cancel."
  - Password text field for confirmation (obscured, with lock icon)
  - Cancel / Delete My Account buttons
  - Loading state during API call (CircularProgressIndicator on button)
  - Calls DELETE /users/me with password in body
  - On success: logout via AuthBloc + deletion confirmation dialog
  - On failure: error SnackBar
- Deletion confirmation dialog: informs user about 30-day grace period + OK button navigates to login
- `ApiConstants.deleteAccount` added for DELETE /users/me

**S8-09: Analytics event SDK integration (Flutter)**

*Core service:*
- `AnalyticsService` at `core/analytics/analytics_service.dart`:
  - In-memory event buffer (List<Map<String, dynamic>>)
  - Periodic flush via Timer.periodic every 30 seconds
  - Force flush when buffer reaches 100 events
  - `onAppPaused()` for immediate flush on app background
  - `flush()` batches events to POST /analytics/events
  - Silent failure (debugPrint only) -- never blocks UI
  - Event names as static constants: submission_started, submission_completed, ai_review_viewed, story_viewed, story_confirmed, story_liked, story_shared, search_performed, feed_scrolled, trending_toggled, delete_account_requested

*Integration points:*
- FeedScreen: submission_started (FAB tap), search_performed (on search), feed_scrolled (scroll depth tracking on dispose), trending_toggled (trending toggle)
- StoryDetailScreen: story_viewed (initState), story_liked (like tap), story_confirmed (confirm tap)
- main.dart: `sl<AnalyticsService>().initialize()` called after dependency registration
- `ApiConstants.analyticsEvents` added for POST /analytics/events
- Registered as lazy singleton in injection.dart

**S9-04: Admin moderation panel UI**

*Data layer:*
- `AdminRemoteDataSource` (abstract + impl): GET /admin/stats, GET /admin/reports, POST /admin/reports/{id}/action, GET /admin/users, POST /admin/users/{id}/action, GET /editorial/analytics
- `AdminRepositoryImpl`: delegates all calls to remote data source

*Domain layer:*
- `AdminStats` entity: totalUsers, totalStories, submissionsToday, pendingReviews, activeReports
- `AdminReport` entity: id, storyId, storyTitle, reporterName, reason, reportedAt, status, storyContent
- `AdminUser` entity: id, name, email, creatorLevel, reputationPoints, storiesCount, status, suspendedUntil
- `EditorAnalytics` entity: pendingQueueSize, avgApprovalTimeToday/Week, rejectionRate, publishedToday, topCities (List<CityContribution>)
- `AdminRepository` (abstract): getDashboardStats, getReports, actionReport, getUsers, actionUser, getEditorAnalytics

*Presentation - AdminBloc:*
- Events: LoadDashboardStats, LoadReports(status), ActionReport(reportId, action), LoadUsers(filter), ActionUser(userId, action, suspendDays)
- State: isLoadingStats/Reports/Users, stats, reports list, users list, userFilter, errorMessage, successMessage
- ActionReport removes actioned report from list optimistically
- ActionUser triggers user list reload

*UI - AdminDashboardScreen:*
- 5 stat cards in 2-column Wrap: Total Users (blue), Total Stories (green), Submissions Today (amber), Pending Reviews (orange), Active Reports (red)
- Quick links section: Reports Queue, User Management, KYC Queue
- Pull-to-refresh, error SnackBar listener

*UI - AdminReportsScreen:*
- List of pending reports with ExpansionTile per report
- Report info: story title, reporter name, reason, reported date (timeAgo), optional story content preview
- 4 action buttons: Dismiss, Remove Story, Warn Creator, Suspend Creator
- Each action shows confirmation dialog before executing
- Success/error SnackBars, pull-to-refresh

*UI - AdminUsersScreen:*
- Filter chips: All, Flagged, Suspended, Pending Deletion (horizontal scroll)
- User cards showing: name, email, status badge (color-coded), creator level, reputation, stories count
- 3 action buttons: Warn (orange), Suspend (red, with duration picker dialog 1/3/7/14/30/90 days), Ban (dark red)
- Confirmation dialogs for warn and ban
- Success/error SnackBars, pull-to-refresh

*Admin route guard:*
- `/admin`, `/admin/reports`, `/admin/users` routes require admin role
- Checked in GoRouter redirect: user.badges must contain 'admin'
- Non-admin users redirected to /feed

**S9-05: Editor analytics dashboard UI**

*Presentation - EditorAnalyticsBloc:*
- Events: LoadEditorAnalytics, FilterEditorAnalyticsByCity(city)
- State: isLoading, EditorAnalytics data, cityFilter, errorMessage

*UI - EditorAnalyticsScreen:*
- City filter dropdown (All Cities + 10 major cities)
- 5 metric cards in 2-column Wrap: Pending Queue (amber), Avg Time Today (blue), Avg Time Week (blue), Rejection Rate (red), Published Today (green)
- Top Contributing Cities bar chart using Container widths (proportional to max count)
- Each bar shows city name, colored bar, and count
- Pull-to-refresh, error SnackBar

*Editorial Queue integration:*
- Added analytics icon button in EditorialQueueScreen app bar
- Navigates to /editorial/analytics
- Route registered in GoRouter with EditorAnalyticsBloc provider

**S9-06: Trending stories UI**

*Feed changes:*
- Added `ToggleTrending` event to FeedBloc
- FeedState extended with `isTrending` bool
- FeedBloc._onToggleTrending: toggles trending, reloads feed with ?trending=true
- All feed load/refresh/filter methods pass trending parameter through
- FeedRepository/DataSource: `trending` parameter added to getFeed()
- GetFeedUseCase: `trending` parameter passed through

*UI changes:*
- Trending fire icon button added to FeedScreen app bar (orange when active)
- App bar subtitle shows "Trending" when active, combined with city filter
- StoryCard: new `showTrendingBadge` parameter
- Trending badge: orange pill with fire icon + "Trending" text, positioned top-left on cover image
- Empty trending state: "Nothing trending right now" with fire icon and "Show All Stories" button

**S9-07: Accessibility audit and fixes**

*Settings screen:*
- Section headers wrapped in Semantics(header: true)
- Notification toggles have Semantics(toggled, label, hint)
- Language tiles have selection state in label
- Delete Account button has Semantics(button, label)
- Withdraw Consent and Download My Data have Semantics(button, hint)
- Password field has Semantics(label, hint)

*Feed screen:*
- FAB has Semantics(button, label: "Submit a new story")
- Search bar close button has Semantics(button, label)
- Search text field has Semantics(label, hint)
- Clear search button has tooltip
- Trending toggle has detailed Semantics label (state-aware)
- City filter has Semantics with current filter state
- Decorative empty state icons wrapped in ExcludeSemantics
- Recent search items have Semantics(label)
- Error icon has Semantics(label: "Error occurred")

*Story card:*
- Entire card wrapped in Semantics with story title, creator name, and trending status
- Cover images: real images have Semantics(image, label), placeholders have ExcludeSemantics
- Trending badge has Semantics(label: "Trending story")

*Story detail:*
- Hero cover image: real image has Semantics(image, label), placeholder has ExcludeSemantics
- Verification badges: "AI Verified" and "Editor Verified" chips have descriptive Semantics labels
- More options button has Semantics(button, label) and tooltip

*Profile screen:*
- App bar title wrapped in Semantics(header: true)
- Settings button has Semantics(button, label)
- Blocked Creators button has Semantics(button, label)

*Editorial queue:*
- App bar title wrapped in Semantics(header: true)
- Analytics icon button has Semantics(button, label) and tooltip
- Thumbnails: real images have Semantics(image, label), placeholders have ExcludeSemantics

*Admin screens:*
- All app bar titles wrapped in Semantics(header: true)
- Dashboard stat cards have Semantics(label) with value
- Quick link tiles have Semantics(hint, button)
- Report cards have Semantics(label) with full context
- Report flag icon has Semantics(label)
- Action buttons have Tooltips
- User cards have Semantics(label) with all user info
- Status badges have Semantics(label)
- Filter chips have selection state in label
- Empty state decorative icons wrapped in ExcludeSemantics
- Suspension duration picker has Semantics(label)

*Editor analytics:*
- City filter dropdown has Semantics(label)
- Metric cards have Semantics(label) with value
- Bar chart entries have Semantics(label) with city and count
- Statistics area has Semantics(label)

**Infrastructure updates:**
- `ApiConstants`: added deleteAccount, analyticsEvents, adminStats, adminReports, adminReportAction(id), adminUsers, adminUserAction(id), editorAnalytics
- `AppRouter`: added adminDashboard, adminReports, adminUsers, editorAnalytics routes + AppRoutes constants; admin routes guarded by badges.contains('admin')
- `injection.dart`: registered AnalyticsService, AdminRemoteDataSource, AdminRepository, AdminBloc, EditorAnalyticsBloc
- `main.dart`: AnalyticsService.initialize() called after configureDependencies()
- `flutter analyze` passes with 0 issues
