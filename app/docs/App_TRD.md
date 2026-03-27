# App Technical Requirements Document (App_TRD)
##Muti Agent- Flutter Frontend

**Version:** 0.1.0
**Parent:** Master_TRD v0.1.0
**Last Updated:** 2026-03-27

---

## 1. Architecture

### Clean Architecture Layers

```
lib/
├── core/           # Shared utilities, theme, constants
├── features/
│   ├── auth/
│   │   ├── data/          # Repositories, data sources, models
│   │   ├── domain/        # Entities, use cases, repo interfaces
│   │   └── presentation/  # BLoC, screens, widgets
│   ├── submission/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   ├── feed/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   ├── editorial/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   ├── community/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   └── profile/
│       ├── data/
│       ├── domain/
│       └── presentation/
└── injection.dart  # Dependency injection setup
```

### State Management: BLoC Pattern
- Each feature has dedicated BLoC(s)
- Events → BLoC → States pattern
- Cubits for simple state (e.g., toggle like)

### Dependencies
| Package | Purpose |
|---------|---------|
| flutter_bloc | State management |
| dio | HTTP client |
| go_router | Navigation |
| get_it + injectable | DI |
| freezed + json_serializable | Model generation |
| image_picker | Photo capture/selection |
| cached_network_image | Image caching |

---

## 2. Feature Specs (from Master_TRD)

### 2.1 Auth (TRD-AUTH-001 to 004)
- Login screen with email/password
- Token storage in secure storage
- Auto-refresh on 401
- Creator level display in profile

### 2.2 Submission (TRD-SUB-001 to 005)
- Create news form: title, cover image (camera/gallery), description, city picker
- Multi-image upload with progress indicator
- "Posted by You" tab with status filter chips
- Status badge colors: yellow (in_progress), blue (under_review), green (published), red (rejected)

### 2.3 AI Preview (TRD-AI-002)
- Side-by-side view: original text vs AI-corrected text
- Accept/reject AI suggestion
- Tag suggestions as chips

### 2.4 Editorial Dashboard (TRD-ED-001 to 003)
- Queue list sorted by priority
- Detail view with original + AI draft + safety scores
- Action buttons: Approve, Edit, Reject, Mark Correction
- City filter dropdown

### 2.5 Feed (TRD-FEED-001 to 005)
- "News By You" tab in main navigation
- Story card: image, title, creator info, likes, confirmations, confirm button
- Infinite scroll with cursor pagination
- City filter + trending toggle
- Pull-to-refresh

### 2.6 Community (TRD-COM-001 to 004)
- Confirm button with counter animation
- Like button with animation
- Confirmation badges ("Confirmed by N users")

### 2.7 Creator Profile (TRD-REP-001 to 004)
- Profile header: avatar, name, level badge, points
- Published stories grid
- Badge showcase
- Accuracy rate display

---

## 3. API Integration
- All models auto-generated from `contract.yaml` schemas
- Dio interceptor handles JWT attach + 401 refresh
- Base URL configurable per environment

---

## 4. Testing Strategy
| Type | Coverage Target | Tool |
|------|----------------|------|
| Widget tests | All screens | flutter_test |
| BLoC tests | All BLoCs | bloc_test |
| Integration tests | Critical flows | integration_test |
| Golden tests | Key UI components | golden_toolkit |
