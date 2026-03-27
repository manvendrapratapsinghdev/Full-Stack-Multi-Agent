# Frontend Stack Skills (Flutter/Dart)

Skills used by the Flutter frontend agents.

---

## Flutter Requirement Agent Skills

| Skill | Description | Tools |
|-------|-------------|-------|
| `openapi_codegen` | Auto-generate Dart models from contract.yaml | openapi_generator_dart_dio |
| `freezed_modeling` | Immutable data classes + JSON serialization | Freezed, json_serializable |
| `contract_validation` | Validate contract changes, detect breaks | openapi_diff |
| `feature_mapping` | Map API endpoints to feature modules | Manual architecture mapping |
| `ui_cataloging` | Component inventory for design system | Widgetbook |
| `platform_tagging` | Tag features for iOS/Android/Web specifics | Custom tagging |

## Flutter Dev Lead Skills

| Skill | Description | Tools |
|-------|-------------|-------|
| `state_management` | BLoC pattern for reactive state | flutter_bloc |
| `navigation` | Declarative routing with deep links | GoRouter |
| `http_client` | Type-safe API calls with interceptors | Dio + Retrofit |
| `local_storage` | SQLite + secure storage for tokens/drafts | Drift, flutter_secure_storage |
| `dependency_injection` | Service locator pattern | GetIt + Injectable |
| `code_generation` | Immutable models + JSON serialization | Freezed, build_runner |
| `design_system` | Material 3 theming + responsive layouts | flutter_screenutil |
| `clean_architecture` | Feature-first data/domain/presentation layers | Manual enforcement |
| `offline_support` | Draft saving + sync when online | Drift SQLite |

## Flutter Code Review Skills

| Skill | Description | Tools |
|-------|-------------|-------|
| `architecture_review` | Clean Architecture boundaries, feature isolation | custom_lint |
| `security_review` | Secure token storage, no hardcoded keys, cert pinning | Manual review |
| `accessibility_review` | WCAG 2.1 AA, 48x48dp tap targets, color contrast | Accessibility scanner |
| `performance_review` | const constructors, ListView.builder, image caching | dart_code_metrics |
| `quality_review` | Freezed usage, null safety, naming conventions | dart analyze |

## Flutter QA Skills

| Skill | Description | Tools |
|-------|-------------|-------|
| `unit_testing` | BLoC + service logic tests | flutter_test, bloc_test |
| `widget_testing` | Component render + interaction tests | flutter_test, mocktail |
| `integration_testing` | Full E2E flows on real devices | Patrol |
| `golden_testing` | Visual regression / snapshot tests | alchemist |
| `static_analysis` | Lint + analyze for code quality | dart analyze, custom_lint |
| `ci_testing` | Automated test runs in CI | Firebase Test Lab |
| `coverage_tracking` | 80%+ business logic coverage | flutter_test --coverage |

**CI Pipeline:** `lint -> analyze -> unit test -> widget test -> integration test -> deploy`

## Flutter Commit Agent Skills

| Skill | Description | Tools |
|-------|-------------|-------|
| `conventional_commits` | Enforce feat/fix/refactor/test prefixes | Manual enforcement |
| `pre_commit_pipeline` | Format + analyze + test before commit | dart format, dart analyze |
| `branch_management` | Feature branch creation and PR | GitHub CLI (gh) |
| `changelog_generation` | Generate release notes from commits | cider |
| `screenshot_capture` | Attach UI screenshots for review | Manual / CI |

**Gate Rule:** NO COMMIT without BOTH Code Review PASS AND QA PASS.

**Pre-commit:** `dart format --set-exit-if-changed -> dart analyze --fatal-infos -> flutter test`
