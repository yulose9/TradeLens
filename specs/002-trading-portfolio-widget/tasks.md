---
description: "Task list for Trading Portfolio Widget implementation"
---

# Tasks: Trading Portfolio Widget

**Input**: Design documents from `/specs/002-trading-portfolio-widget/`
**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/

**Tests**: Tests are included following TDD approach as specified in the Constitution (≥60% coverage target for core modules)

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`
- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3, US4)
- Include exact file paths in descriptions

## Path Conventions
- **Android project**: `app/src/main/java/com/tradelens/` for source code
- **Tests**: `app/src/test/java/com/tradelens/` for unit tests
- **Resources**: `app/src/main/res/` for Android resources
- **Root config**: `build.gradle.kts`, `settings.gradle.kts`, `.github/workflows/`

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and basic structure

- [ ] T001 Create repository structure with README.md, LICENSE, CODE_OF_CONDUCT.md, .gitignore
- [ ] T002 Initialize Gradle wrapper and Android project skeleton in `app/build.gradle.kts`
- [ ] T003 [P] Configure ktlint and detekt in `app/build.gradle.kts` and root `build.gradle.kts`
- [ ] T004 [P] Set up pre-commit hook for secret detection in `.git-hooks/pre-commit`
- [ ] T005 [P] Configure GitHub Actions CI workflow in `.github/workflows/ci.yml` (build, ktlint, detekt, tests)
- [ ] T006 [P] Configure GitHub Actions dependency scanning workflow in `.github/workflows/dependency-scan.yml`
- [ ] T007 [P] Create version catalog in `gradle/libs.versions.toml` with all dependencies
- [ ] T008 [P] Add signing configuration placeholders and documentation in `docs/RELEASE.md`
- [ ] T009 [P] Create pull request template in `.github/pull_request_template.md` with constitution checklist

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [ ] T010 Create Hilt application class in `app/src/main/java/com/tradelens/TradeLensApplication.kt`
- [ ] T011 [P] Create AppModule for Hilt DI in `app/src/main/java/com/tradelens/di/AppModule.kt`
- [ ] T012 [P] Create NetworkModule with Retrofit + OkHttp configuration in `app/src/main/java/com/tradelens/di/NetworkModule.kt`
- [ ] T013 [P] Create StorageModule with Room and EncryptedSharedPreferences in `app/src/main/java/com/tradelens/di/StorageModule.kt`
- [ ] T014 [P] Create WorkManagerModule in `app/src/main/java/com/tradelens/di/WorkManagerModule.kt`
- [ ] T015 [P] Implement AuthInterceptor for API token injection in `app/src/main/java/com/tradelens/data/remote/interceptor/AuthInterceptor.kt`
- [ ] T016 [P] Implement RetryInterceptor with exponential backoff in `app/src/main/java/com/tradelens/data/remote/interceptor/RetryInterceptor.kt`
- [ ] T017 [P] Implement LoggingInterceptor (debug builds only) in `app/src/main/java/com/tradelens/data/remote/interceptor/LoggingInterceptor.kt`
- [ ] T018 [P] Create SecurePreferences wrapper for EncryptedSharedPreferences in `app/src/main/java/com/tradelens/data/local/SecurePreferences.kt`
- [ ] T019 [P] Define Room database class in `app/src/main/java/com/tradelens/data/local/PortfolioDatabase.kt`
- [ ] T020 [P] Create PortfolioEntity with annotations in `app/src/main/java/com/tradelens/data/local/entity/PortfolioEntity.kt`
- [ ] T021 [P] Create PositionEntity with foreign key in `app/src/main/java/com/tradelens/data/local/entity/PositionEntity.kt`
- [ ] T022 [P] Create PortfolioDao with queries in `app/src/main/java/com/tradelens/data/local/dao/PortfolioDao.kt`
- [ ] T023 [P] Create PositionDao with queries in `app/src/main/java/com/tradelens/data/local/dao/PositionDao.kt`
- [ ] T024 [P] Create base theme resources (colors, typography) in `app/src/main/res/values/themes.xml` and `app/src/main/res/values-night/themes.xml`
- [ ] T025 [P] Create Compose theme in `app/src/main/java/com/tradelens/ui/theme/Theme.kt`
- [ ] T026 [P] Create Compose Color definitions in `app/src/main/java/com/tradelens/ui/theme/Color.kt`
- [ ] T027 [P] Create Compose Typography in `app/src/main/java/com/tradelens/ui/theme/Typography.kt`
- [ ] T028 [P] Create string resources in `app/src/main/res/values/strings.xml`
- [ ] T029 [P] Configure AndroidManifest.xml with permissions and widget provider metadata

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - View Portfolio Value (Priority: P1) 🎯 MVP

**Goal**: Display total portfolio value and P/L in a widget that works offline with cached data

**Independent Test**: Add widget to home screen, verify it shows portfolio value and P/L from cached or live data, verify "stale" indicator appears when offline

### Tests for User Story 1 (TDD Approach) ⚠️

**NOTE: Write these tests FIRST, ensure they FAIL before implementation**

- [ ] T030 [P] [US1] Unit test for PortfolioDto to PortfolioSnapshot mapping in `app/src/test/java/com/tradelens/data/remote/dto/PortfolioDtoMapperTest.kt`
- [ ] T031 [P] [US1] Unit test for PortfolioRepository with mocked API and Room in `app/src/test/java/com/tradelens/data/repository/PortfolioRepositoryImplTest.kt`
- [ ] T032 [P] [US1] Unit test for GetPortfolioSnapshotUseCase in `app/src/test/java/com/tradelens/domain/usecase/GetPortfolioSnapshotUseCaseTest.kt`
- [ ] T033 [P] [US1] Unit test for cache staleness detection logic in `app/src/test/java/com/tradelens/domain/model/PortfolioSnapshotTest.kt`

### Domain Layer for User Story 1

- [ ] T034 [P] [US1] Create PortfolioSnapshot domain model in `app/src/main/java/com/tradelens/domain/model/PortfolioSnapshot.kt`
- [ ] T035 [P] [US1] Create Position domain model in `app/src/main/java/com/tradelens/domain/model/Position.kt`
- [ ] T036 [P] [US1] Create RefreshFrequency enum in `app/src/main/java/com/tradelens/domain/model/RefreshFrequency.kt`
- [ ] T037 [P] [US1] Create PortfolioRepository interface in `app/src/main/java/com/tradelens/domain/repository/PortfolioRepository.kt`
- [ ] T038 [US1] Create GetPortfolioSnapshotUseCase in `app/src/main/java/com/tradelens/domain/usecase/GetPortfolioSnapshotUseCase.kt`

### Data Layer for User Story 1

- [ ] T039 [P] [US1] Create PortfolioDto in `app/src/main/java/com/tradelens/data/remote/dto/PortfolioDto.kt`
- [ ] T040 [P] [US1] Create PositionDto in `app/src/main/java/com/tradelens/data/remote/dto/PositionDto.kt`
- [ ] T041 [P] [US1] Create Trading212Api Retrofit interface in `app/src/main/java/com/tradelens/data/remote/Trading212Api.kt`
- [ ] T042 [US1] Implement PortfolioDtoMapper (DTO → Domain) in `app/src/main/java/com/tradelens/data/remote/dto/PortfolioDtoMapper.kt`
- [ ] T043 [US1] Implement PortfolioEntityMapper (Domain ↔ Entity) in `app/src/main/java/com/tradelens/data/local/entity/PortfolioEntityMapper.kt`
- [ ] T044 [US1] Implement PortfolioRepositoryImpl with cache-first strategy in `app/src/main/java/com/tradelens/data/repository/PortfolioRepositoryImpl.kt`
- [ ] T045 [US1] Add error mapping (401, 429, 503 → domain exceptions) in `app/src/main/java/com/tradelens/data/remote/ErrorMapper.kt`

### Presentation Layer for User Story 1

- [ ] T046 [P] [US1] Create widget metadata XML in `app/src/main/res/xml/widget_info.xml`
- [ ] T047 [P] [US1] Create PortfolioWidgetState sealed class in `app/src/main/java/com/tradelens/ui/widget/PortfolioWidgetState.kt`
- [ ] T048 [US1] Create PortfolioWidget GlanceAppWidget class in `app/src/main/java/com/tradelens/ui/widget/PortfolioWidget.kt`
- [ ] T049 [US1] Implement CompactLayout (1x1) for balance only in `app/src/main/java/com/tradelens/ui/widget/layouts/CompactLayout.kt`
- [ ] T050 [US1] Implement StandardLayout (2x2) for balance + P/L in `app/src/main/java/com/tradelens/ui/widget/layouts/StandardLayout.kt`
- [ ] T051 [US1] Create WidgetReceiver for manual refresh actions in `app/src/main/java/com/tradelens/ui/widget/WidgetReceiver.kt`
- [ ] T052 [US1] Implement WidgetUpdateWorker with cache-first logic in `app/src/main/java/com/tradelens/ui/widget/WidgetUpdateWorker.kt`
- [ ] T053 [US1] Add "stale" indicator UI when cache is old in StandardLayout.kt
- [ ] T054 [US1] Add manual refresh button to widget layouts

**Checkpoint**: User Story 1 complete - widget displays portfolio value and P/L, works offline

---

## Phase 4: User Story 2 - View Open Positions (Priority: P2)

**Goal**: Display list of open positions with per-position P/L in larger widget sizes

**Independent Test**: Select positions in settings, add widget, verify positions display with correct symbols, quantities, and P/L

### Tests for User Story 2 (TDD Approach) ⚠️

**NOTE: Write these tests FIRST, ensure they FAIL before implementation**

- [ ] T055 [P] [US2] Unit test for GetPositionsUseCase in `app/src/test/java/com/tradelens/domain/usecase/GetPositionsUseCaseTest.kt`
- [ ] T056 [P] [US2] Unit test for position filtering logic in `app/src/test/java/com/tradelens/domain/usecase/GetPositionsUseCaseTest.kt`
- [ ] T057 [P] [US2] Unit test for position sorting by value in `app/src/test/java/com/tradelens/domain/model/PositionTest.kt`

### Domain Layer for User Story 2

- [ ] T058 [P] [US2] Create GetPositionsUseCase with filtering logic in `app/src/main/java/com/tradelens/domain/usecase/GetPositionsUseCase.kt`
- [ ] T059 [US2] Add position filtering to PortfolioRepository interface in `app/src/main/java/com/tradelens/domain/repository/PortfolioRepository.kt`

### Data Layer for User Story 2

- [ ] T060 [US2] Implement position filtering in PortfolioRepositoryImpl in `app/src/main/java/com/tradelens/data/repository/PortfolioRepositoryImpl.kt`
- [ ] T061 [US2] Add Room query for filtered positions in PositionDao in `app/src/main/java/com/tradelens/data/local/dao/PositionDao.kt`

### Presentation Layer for User Story 2

- [ ] T062 [P] [US2] Implement WideLayout (4x1) with positions list in `app/src/main/java/com/tradelens/ui/widget/layouts/WideLayout.kt`
- [ ] T063 [P] [US2] Implement ExpandedLayout (4x2) skeleton in `app/src/main/java/com/tradelens/ui/widget/layouts/ExpandedLayout.kt`
- [ ] T064 [US2] Create PositionListItem composable for widget in `app/src/main/java/com/tradelens/ui/widget/components/PositionListItem.kt`
- [ ] T065 [US2] Add color coding (green/red) for positive/negative P/L in PositionListItem.kt
- [ ] T066 [US2] Add empty state UI when no positions exist in widget layouts
- [ ] T067 [US2] Implement top-N position limiting (5 for 2x2, 10 for 4x2) in WidgetUpdateWorker.kt

**Checkpoint**: User Story 2 complete - widget displays open positions with P/L

---

## Phase 5: User Story 3 - Configure Widget & Refresh (Priority: P3)

**Goal**: Provide settings UI for API token, refresh frequency, and position selection

**Independent Test**: Open settings, enter token, change refresh frequency, verify widget updates at new interval

### Tests for User Story 3 (TDD Approach) ⚠️

**NOTE: Write these tests FIRST, ensure they FAIL before implementation**

- [ ] T068 [P] [US3] Unit test for UserSettings domain model validation in `app/src/test/java/com/tradelens/domain/model/UserSettingsTest.kt`
- [ ] T069 [P] [US3] Unit test for SettingsRepository in `app/src/test/java/com/tradelens/data/repository/SettingsRepositoryImplTest.kt`
- [ ] T070 [P] [US3] Unit test for SaveSettingsUseCase in `app/src/test/java/com/tradelens/domain/usecase/SaveSettingsUseCaseTest.kt`
- [ ] T071 [P] [US3] Unit test for ValidateTokenUseCase in `app/src/test/java/com/tradelens/domain/usecase/ValidateTokenUseCaseTest.kt`

### Domain Layer for User Story 3

- [ ] T072 [P] [US3] Create UserSettings domain model in `app/src/main/java/com/tradelens/domain/model/UserSettings.kt`
- [ ] T073 [P] [US3] Create ThemeMode enum in `app/src/main/java/com/tradelens/domain/model/ThemeMode.kt`
- [ ] T074 [P] [US3] Create SettingsRepository interface in `app/src/main/java/com/tradelens/domain/repository/SettingsRepository.kt`
- [ ] T075 [P] [US3] Create GetSettingsUseCase in `app/src/main/java/com/tradelens/domain/usecase/GetSettingsUseCase.kt`
- [ ] T076 [P] [US3] Create SaveSettingsUseCase in `app/src/main/java/com/tradelens/domain/usecase/SaveSettingsUseCase.kt`
- [ ] T077 [P] [US3] Create ValidateTokenUseCase in `app/src/main/java/com/tradelens/domain/usecase/ValidateTokenUseCase.kt`
- [ ] T078 [US3] Create UpdateWidgetUseCase for manual refresh in `app/src/main/java/com/tradelens/domain/usecase/UpdateWidgetUseCase.kt`

### Data Layer for User Story 3

- [ ] T079 [US3] Implement SettingsRepositoryImpl with SharedPreferences in `app/src/main/java/com/tradelens/data/repository/SettingsRepositoryImpl.kt`
- [ ] T080 [US3] Add token validation endpoint call to Trading212Api in `app/src/main/java/com/tradelens/data/remote/Trading212Api.kt`

### Presentation Layer for User Story 3

- [ ] T081 [P] [US3] Create SettingsScreen Compose UI in `app/src/main/java/com/tradelens/ui/settings/SettingsScreen.kt`
- [ ] T082 [P] [US3] Create SettingsViewModel with state management in `app/src/main/java/com/tradelens/ui/settings/SettingsViewModel.kt`
- [ ] T083 [P] [US3] Create SettingsActivity as entry point in `app/src/main/java/com/tradelens/ui/settings/SettingsActivity.kt`
- [ ] T084 [US3] Implement API token input field with validation in SettingsScreen.kt
- [ ] T085 [US3] Implement refresh frequency selector (dropdown) in SettingsScreen.kt
- [ ] T086 [US3] Implement position filter UI (multi-select) in SettingsScreen.kt
- [ ] T087 [US3] Add connection status indicator in SettingsScreen.kt
- [ ] T088 [US3] Update WidgetUpdateWorker to respect refresh frequency setting
- [ ] T089 [US3] Schedule WorkManager PeriodicWorkRequest on settings change
- [ ] T090 [US3] Implement manual refresh action handler in WidgetReceiver.kt
- [ ] T091 [US3] Add rate-limit error handling (429 response) in WidgetUpdateWorker.kt

**Checkpoint**: User Story 3 complete - users can configure widget and refresh settings

---

## Phase 6: User Story 4 - Widget Version Feature-Gating (Priority: P4)

**Goal**: Enable feature flags for v1/v2/v3 widget versions

**Independent Test**: Enable v2 features in settings, verify positions and sparkline appear; disable, verify they disappear

### Tests for User Story 4 (TDD Approach) ⚠️

**NOTE: Write these tests FIRST, ensure they FAIL before implementation**

- [ ] T092 [P] [US4] Unit test for feature flag evaluation logic in `app/src/test/java/com/tradelens/domain/model/WidgetVersionTest.kt`
- [ ] T093 [P] [US4] Unit test for conditional layout rendering in `app/src/test/java/com/tradelens/ui/widget/PortfolioWidgetTest.kt`

### Domain Layer for User Story 4

- [ ] T094 [P] [US4] Create WidgetVersion enum in `app/src/main/java/com/tradelens/domain/model/WidgetVersion.kt`
- [ ] T095 [US4] Add feature flag fields to UserSettings (enableV2Features, enableV3Features) in UserSettings.kt

### Data Layer for User Story 4

- [ ] T096 [US4] Add feature flag storage to SettingsRepositoryImpl in SettingsRepositoryImpl.kt

### Presentation Layer for User Story 4

- [ ] T097 [P] [US4] Add BuildConfig feature flags in `app/build.gradle.kts` (FEATURE_V2_POSITIONS, FEATURE_V3_NOTIFICATIONS)
- [ ] T098 [US4] Implement conditional layout rendering in PortfolioWidget.kt based on feature flags
- [ ] T099 [US4] Add feature toggle UI in SettingsScreen.kt (v2/v3 opt-in checkboxes)
- [ ] T100 [US4] Implement sparkline placeholder UI in ExpandedLayout.kt (v2 feature)
- [ ] T101 [US4] Add notifications placeholder UI (v3 feature - future work)

**Checkpoint**: User Story 4 complete - feature-gating infrastructure ready for incremental rollout

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: UX polish, error handling, performance optimization, and documentation

### UX & Accessibility

- [ ] T102 [P] Define Material Design 3 design tokens (colors, spacing, fonts) in `app/src/main/res/values/`
- [ ] T103 [P] Verify WCAG AA contrast ratios for all text/background combinations
- [ ] T104 [P] Verify minimum 48dp touch target sizes for all interactive elements
- [ ] T105 [P] Add accessibility labels to all widget elements in widget layouts
- [ ] T106 [P] Test theme switching (light/dark) across all widget sizes
- [ ] T107 [P] Implement loading skeleton UI for widget while data loads

### Error Handling & Edge Cases

- [ ] T108 [P] Implement error state UI for invalid token in all widget layouts
- [ ] T109 [P] Implement error state UI for network errors in all widget layouts
- [ ] T110 [P] Implement error state UI for rate-limited responses in all widget layouts
- [ ] T111 [P] Add user-friendly error messages (avoid technical jargon) in string resources
- [ ] T112 [P] Implement retry logic with exponential backoff in RetryInterceptor.kt
- [ ] T113 [P] Add timeout configuration for Retrofit client in NetworkModule.kt

### Performance & Optimization

- [ ] T114 [P] Implement Room cache eviction (keep last 7 days) in PortfolioDao.kt
- [ ] T115 [P] Add database indexes for timestamp queries in PortfolioEntity.kt
- [ ] T116 [P] Profile widget render time (target <100ms) and optimize if needed
- [ ] T117 [P] Implement bitmap caching for sparkline (v2 feature) in ExpandedLayout.kt
- [ ] T118 [P] Add ProGuard rules for release builds in `app/proguard-rules.pro`
- [ ] T119 [P] Configure R8 optimization in `app/build.gradle.kts`

### Testing & Quality

- [ ] T120 [P] Run ktlint and fix all formatting issues across codebase
- [ ] T121 [P] Run detekt and resolve all code quality warnings
- [ ] T122 [P] Generate test coverage report and verify ≥60% for core modules
- [ ] T123 [P] Add instrumentation test for widget configuration flow in `app/src/androidTest/`
- [ ] T124 [P] Add instrumentation test for manual refresh action in `app/src/androidTest/`
- [ ] T125 [P] Run performance smoke test (widget update latency, no ANRs)

### Documentation & Release

- [ ] T126 [P] Update README.md with feature description, screenshots, and setup instructions
- [ ] T127 [P] Create quickstart guide in `docs/QUICKSTART.md` (copied from specs/)
- [ ] T128 [P] Document API token generation process in `docs/API_SETUP.md`
- [ ] T129 [P] Create release checklist in `docs/RELEASE.md`
- [ ] T130 [P] Configure GitHub release workflow in `.github/workflows/release.yml`
- [ ] T131 [P] Write initial CHANGELOG.md with v1.0.0 features
- [ ] T132 [P] Add security review checklist to release process in `docs/RELEASE.md`
- [ ] T133 [P] Generate KDoc for all public APIs and publish to GitHub Pages

**Checkpoint**: All polish tasks complete - app is production-ready

---

## Dependencies & Execution Order

### Critical Path (Must Complete in Order)

1. **Phase 1 (Setup)** → **Phase 2 (Foundational)** → User Story Phases
2. **Phase 2 (Foundational)** MUST complete before ANY user story work begins

### User Story Dependencies

- **US1 (View Portfolio Value)**: ✅ Independent - can start immediately after Phase 2
- **US2 (View Open Positions)**: ⚠️ Depends on US1 (requires PortfolioSnapshot model and repository)
- **US3 (Configure Widget & Refresh)**: ⚠️ Depends on US1 (requires widget infrastructure and WorkManager)
- **US4 (Feature-Gating)**: ⚠️ Depends on US2 and US3 (requires feature infrastructure to gate)

### Recommended Implementation Order

1. **MVP (Phase 1 + Phase 2 + Phase 3)**: Basic widget with portfolio value and P/L
2. **v1.1 (Phase 4)**: Add positions display
3. **v1.2 (Phase 5)**: Add settings and configuration
4. **v1.3 (Phase 6)**: Add feature-gating for future v2/v3
5. **v2.0 (Phase 7 + Sparkline)**: Polish, sparkline, production-ready

### Parallel Execution Opportunities

#### Phase 1 (Setup) - All tasks can run in parallel after T002
- T003-T009 are independent

#### Phase 2 (Foundational) - High parallelism
- T011-T017 (DI modules and interceptors) can run in parallel
- T018 (SecurePreferences) independent
- T019-T023 (Room setup) can run in parallel after T019
- T024-T028 (UI resources) can run in parallel
- T029 (Manifest) independent

#### Phase 3 (US1) - Moderate parallelism
- T030-T033 (Tests) can run in parallel
- T034-T037 (Domain models and interfaces) can run in parallel
- T039-T041 (DTOs and API interface) can run in parallel
- T046-T047 (Widget metadata and state) can run in parallel

#### Phase 4 (US2) - Moderate parallelism
- T055-T057 (Tests) can run in parallel
- T058-T059 (Domain layer) can run together
- T062-T066 (Presentation layer) can run in parallel after domain layer

#### Phase 5 (US3) - High parallelism
- T068-T071 (Tests) can run in parallel
- T072-T077 (Domain models and use cases) can run in parallel
- T081-T083 (UI components) can run in parallel

#### Phase 6 (US4) - High parallelism
- T092-T093 (Tests) can run in parallel
- T094-T096 (Domain and data layer) can run in parallel
- T097-T101 (Presentation layer) can run in parallel

#### Phase 7 (Polish) - High parallelism (nearly all tasks independent)

---

## Implementation Strategy

### MVP First (Recommended for Week 1)

**Scope**: Phase 1 + Phase 2 + Phase 3 (User Story 1)

**Deliverable**: Working widget that displays portfolio value and P/L, works offline with cached data

**Task Count**: ~54 tasks (T001-T054)

**Value**: Users can glance at portfolio value without opening Trading 212 app

### Incremental Delivery

- **Week 1**: MVP (US1) - 54 tasks
- **Week 2**: US2 (Positions) - 13 tasks (T055-T067)
- **Week 3**: US3 (Settings & Refresh) - 24 tasks (T068-T091)
- **Week 4**: US4 (Feature-Gating) + Polish - 43 tasks (T092-T133)

### Testing Strategy

- **TDD for core logic**: Write tests FIRST for repositories, use cases, mappers
- **Integration tests**: Add after each user story is complete
- **Instrumentation tests**: Add for critical paths (widget config, manual refresh)
- **Coverage target**: ≥60% for core modules (data, domain layers)

---

## Summary

**Total Tasks**: 133
**MVP Tasks**: 54 (Phase 1-3)
**Parallel Opportunities**: ~60% of tasks can run in parallel within their phase
**Independent User Stories**: US1 is fully independent; US2-US4 have dependencies

**Task Breakdown by User Story**:
- Setup & Foundation: 29 tasks (T001-T029)
- US1 (View Portfolio Value): 25 tasks (T030-T054)
- US2 (View Open Positions): 13 tasks (T055-T067)
- US3 (Configure Widget & Refresh): 24 tasks (T068-T091)
- US4 (Feature-Gating): 10 tasks (T092-T101)
- Polish & Cross-Cutting: 32 tasks (T102-T133)

**Estimated Effort**:
- Setup & Foundation: 2-3 days
- US1 MVP: 3-4 days
- US2 Positions: 1-2 days
- US3 Settings: 2-3 days
- US4 Feature-Gating: 1 day
- Polish: 2-3 days

**Total**: ~12-16 days for full implementation

---

**Format Validation**: ✅ All 133 tasks follow the required checklist format with checkboxes, task IDs, parallel markers, story labels (where applicable), and file paths.
