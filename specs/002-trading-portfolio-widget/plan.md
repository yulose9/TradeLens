# Implementation Plan: Trading Portfolio Widget

**Branch**: `002-trading-portfolio-widget` | **Date**: 2025-10-21 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/002-trading-portfolio-widget/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command. See `.specify/templates/commands/plan.md` for the execution workflow.

## Summary

Build an Android widget application that displays Trading 212 portfolio information via Context7 API wrapper. The widget shows total portfolio value, profit/loss (absolute and percentage), and open positions with per-position P/L. Supports multiple widget sizes (1x1, 2x2, 4x1, 4x2), configurable refresh intervals (1-2 min fast, 5-15 min normal, 30+ min battery-saver), offline caching, and feature-gated versions (v1: balance+P/L, v2: +positions+sparkline, v3: +notifications). Built with Kotlin, Jetpack Compose, Glance API for widgets, WorkManager for background updates, Hilt for DI, Retrofit for API calls, EncryptedSharedPreferences for secrets, and Room/DataStore for caching.

## Technical Context

**Language/Version**: Kotlin (JVM), Android API 26+ (Android 8.0 Oreo minimum for EncryptedSharedPreferences, Glance support)  
**Primary Dependencies**: 
  - UI: Jetpack Compose (in-app screens), Glance API (widget rendering)
  - Networking: Retrofit 2.9+, OkHttp 4.x (interceptors for auth, logging, retry)
  - JSON: Kotlinx.serialization (consistent with Kotlin-first approach)
  - Background: WorkManager 2.8+ (periodic and one-off widget updates)
  - Storage: EncryptedSharedPreferences (secrets), Room 2.5+ or DataStore Proto (cached snapshots)
  - DI: Hilt 2.48+ (Dagger-based, Android-optimized)
  - Testing: JUnit 4.13+, MockK 1.13+, Robolectric 4.10+ (ViewModel/Compose tests), Espresso (instrumentation)
  - Static Analysis: ktlint 0.50+, detekt 1.23+
  - CI/CD: GitHub Actions (build, test, static analysis, dependency scanning via Dependabot)
  - API: Context7 MCP wrapper for Trading 212 API Beta

**Storage**: 
  - EncryptedSharedPreferences: API tokens, user credentials
  - Room or DataStore Proto: Cached PortfolioSnapshot entities (value, P/L, positions, timestamp)
  - File-based cache for sparkline data points (optional, Phase 2)

**Testing**: 
  - JUnit + MockK for unit tests (repositories, use cases, ViewModels)
  - Robolectric for Android-dependent unit tests (Compose UI logic, ViewModels)
  - Espresso for instrumentation tests (critical user journeys: widget config, manual refresh)
  - Target ≥60% coverage on core modules (data layer, domain layer, API client)

**Target Platform**: Android 8.0+ (API 26+), targeting latest stable API (34+) for development  
**Project Type**: Mobile (native Android, single-module initially, multi-module future consideration)  
**Performance Goals**: 
  - Widget update < 100ms for UI rendering
  - API response handling < 500ms (network dependent)
  - Background work respects battery: no ANRs, exponential backoff on failures
  - Memory: < 50MB resident for widget service, < 100MB for full app

**Constraints**: 
  - Offline-capable: must display cached data when network unavailable
  - Rate-limit aware: respect Trading 212 API limits, exponential backoff
  - Battery-conscious: configurable refresh intervals, WorkManager constraints
  - Secure: no secrets in source control, encrypted storage for tokens
  - Privacy: no off-device transmission of portfolio data without explicit user consent

**Scale/Scope**: 
  - Single-user app (no backend, no user management)
  - 1-5 widgets per user device
  - 10-100 positions per portfolio snapshot
  - Minimal UI: 2-3 screens (settings, about, token management)
  - ~5K-10K lines of Kotlin code (estimated)

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

Verify compliance with TradeLens Constitution (`.specify/memory/constitution.md`):

- [x] **Code Quality**: Plan includes code review checkpoints (PR template), static analysis integration (ktlint + detekt in CI pipeline), KDoc for public APIs, modular architecture (presentation/domain/data layers), dependency version pinning in build.gradle.kts
- [x] **Testing Standards**: Unit test coverage target (≥60% for core modules: data transformation, API parsing, widget update logic) documented, test-first approach for core logic (TDD for repositories, use cases), integration tests for API client + WorkManager jobs
- [x] **Security**: No secrets in spec/plan (✓ verified), encryption strategy documented (EncryptedSharedPreferences for tokens, Room for cache), pre-commit hook planned (git-secrets/truffleHog pattern detection), signing keys in GitHub Secrets, no secrets in logs
- [x] **UX Consistency**: Design follows Material Design 3 and Android widget design guidelines (colors, typography, spacing, icons), theme support (light/dark) planned (theme-aware resource qualifiers), widget sizes (1×1, 2×1, 2×2, 4×1, 4×2) addressed in Glance layouts, accessibility (sp units, 48dp touch targets, WCAG AA contrast), loading/error/empty states defined
- [x] **Performance**: Background work rate-limiting strategy defined (WorkManager periodic with user-configurable intervals: fast 1-2min, normal 5-15min, battery-saver 30+min, exponential backoff on failures), API caching approach documented (Room/DataStore with timestamp-based expiration, optimistic cache-first display), UI responsiveness targets (<100ms widget render, <500ms API response handling), memory efficiency (minimize allocations, scale bitmaps for sparkline)
- [x] **Privacy**: Data transmission policy defined (on-device by default, no off-device transmission without explicit user consent), user opt-out mechanism planned (settings toggle for optional telemetry, off by default), privacy-first logging (debug logs only in debug builds)
- [x] **CI/CD**: Automated quality gates planned (GitHub Actions: build, ktlint, detekt, unit tests, Dependabot dependency scanning, signed artifact generation via GitHub Secrets)
- [x] **Release Criteria**: Changelog requirement documented, migration guide planned (if storage schema changes across versions), security review checkpoint (pre-release secret scan + manual review), performance smoke test (widget update latency, ANR check)

## Project Structure

### Documentation (this feature)

```
specs/[###-feature]/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)

```
TradeLens/
├── app/                              # Main Android application module
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/com/tradelens/
│   │   │   │   ├── TradeLensApplication.kt      # Hilt app entry point
│   │   │   │   ├── di/                          # Hilt modules
│   │   │   │   │   ├── AppModule.kt
│   │   │   │   │   ├── NetworkModule.kt
│   │   │   │   │   ├── StorageModule.kt
│   │   │   │   │   └── WorkManagerModule.kt
│   │   │   │   ├── ui/                          # Compose UI + widget
│   │   │   │   │   ├── widget/
│   │   │   │   │   │   ├── PortfolioWidget.kt  # Glance widget composable
│   │   │   │   │   │   ├── WidgetReceiver.kt   # BroadcastReceiver for actions
│   │   │   │   │   │   ├── WidgetUpdateWorker.kt
│   │   │   │   │   │   └── layouts/             # Size-specific layouts
│   │   │   │   │   │       ├── CompactLayout.kt    # 1x1
│   │   │   │   │   │       ├── StandardLayout.kt   # 2x2
│   │   │   │   │   │       ├── WideLayout.kt       # 4x1
│   │   │   │   │   │       └── ExpandedLayout.kt   # 4x2 with sparkline
│   │   │   │   │   ├── settings/
│   │   │   │   │   │   ├── SettingsActivity.kt
│   │   │   │   │   │   ├── SettingsScreen.kt   # Compose UI
│   │   │   │   │   │   └── SettingsViewModel.kt
│   │   │   │   │   └── theme/
│   │   │   │   │       ├── Theme.kt             # Compose + Glance theme
│   │   │   │   │       ├── Color.kt
│   │   │   │   │       └── Typography.kt
│   │   │   │   ├── domain/                      # Business logic
│   │   │   │   │   ├── model/
│   │   │   │   │   │   ├── PortfolioSnapshot.kt
│   │   │   │   │   │   ├── Position.kt
│   │   │   │   │   │   └── UserSettings.kt
│   │   │   │   │   ├── usecase/
│   │   │   │   │   │   ├── GetPortfolioSnapshotUseCase.kt
│   │   │   │   │   │   ├── GetPositionsUseCase.kt
│   │   │   │   │   │   ├── RefreshTokenUseCase.kt
│   │   │   │   │   │   └── UpdateWidgetUseCase.kt
│   │   │   │   │   └── repository/              # Repository interfaces
│   │   │   │   │       ├── PortfolioRepository.kt
│   │   │   │   │       └── SettingsRepository.kt
│   │   │   │   └── data/                        # Data layer implementations
│   │   │   │       ├── remote/
│   │   │   │       │   ├── Trading212Api.kt     # Retrofit interface
│   │   │   │       │   ├── Context7ApiClient.kt # Wrapper with interceptors
│   │   │   │       │   ├── dto/                 # API response models
│   │   │   │       │   │   ├── PortfolioDto.kt
│   │   │   │       │   │   └── PositionDto.kt
│   │   │   │       │   └── interceptor/
│   │   │   │       │       ├── AuthInterceptor.kt
│   │   │   │       │       ├── RetryInterceptor.kt
│   │   │   │       │       └── LoggingInterceptor.kt
│   │   │   │       ├── local/
│   │   │   │       │   ├── PortfolioDatabase.kt # Room database
│   │   │   │       │   ├── dao/
│   │   │   │       │   │   ├── PortfolioDao.kt
│   │   │   │       │   │   └── PositionDao.kt
│   │   │   │       │   ├── entity/
│   │   │   │       │   │   ├── PortfolioEntity.kt
│   │   │   │       │   │   └── PositionEntity.kt
│   │   │   │       │   └── SecurePreferences.kt # EncryptedSharedPreferences wrapper
│   │   │   │       └── repository/              # Repository implementations
│   │   │   │           ├── PortfolioRepositoryImpl.kt
│   │   │   │           └── SettingsRepositoryImpl.kt
│   │   │   ├── res/
│   │   │   │   ├── values/
│   │   │   │   │   ├── strings.xml
│   │   │   │   │   ├── themes.xml
│   │   │   │   │   └── colors.xml
│   │   │   │   ├── values-night/                # Dark theme overrides
│   │   │   │   │   ├── themes.xml
│   │   │   │   │   └── colors.xml
│   │   │   │   ├── xml/
│   │   │   │   │   └── widget_info.xml          # Widget provider metadata
│   │   │   │   └── drawable/                    # Icons, assets
│   │   │   └── AndroidManifest.xml
│   │   └── test/                                # Unit tests
│   │       └── java/com/tradelens/
│   │           ├── domain/
│   │           │   └── usecase/
│   │           │       ├── GetPortfolioSnapshotUseCaseTest.kt
│   │           │       └── GetPositionsUseCaseTest.kt
│   │           └── data/
│   │               ├── repository/
│   │               │   └── PortfolioRepositoryImplTest.kt
│   │               └── remote/
│   │                   └── Trading212ApiTest.kt
│   └── build.gradle.kts                         # App module build config
├── build.gradle.kts                             # Root build config
├── settings.gradle.kts
├── gradle.properties
├── gradle/
│   └── libs.versions.toml                       # Version catalog
├── .github/
│   └── workflows/
│       ├── ci.yml                               # CI pipeline
│       └── release.yml                          # Release pipeline
├── .git-hooks/
│   └── pre-commit                               # Secret detection hook
├── detekt.yml                                   # Detekt config
├── .editorconfig                                # ktlint config
└── README.md                                    # Developer guide
```

**Structure Decision**: Native Android single-module architecture following clean architecture principles (presentation/domain/data layers). Widget code lives in `ui/widget/` using Glance API. Hilt provides dependency injection across layers. Repository pattern abstracts data sources (remote API via Retrofit, local cache via Room). WorkManager handles background refresh. This structure supports future multi-module refactoring if app grows beyond widget scope.

## Complexity Tracking

*Fill ONLY if Constitution Check has violations that must be justified*

No violations detected. All constitutional principles are satisfied:
- Code quality enforced via static analysis and review process
- Testing standards met with ≥60% coverage target for core modules
- Security addressed via encrypted storage and secret scanning
- UX consistency planned with theme support and accessibility standards
- Performance optimized with caching, rate-limiting, and responsiveness targets
- Privacy protected with on-device-first approach and user consent

---

## Phase 0: Research (COMPLETED)

**Status**: ✅ Complete
**Output**: [research.md](./research.md)

### Key Decisions Made:
1. **Widget Rendering**: Jetpack Glance API (modern, Compose-like)
2. **JSON Library**: Kotlinx.serialization (Kotlin-first, zero-reflection)
3. **Cache Storage**: Room database (relational data, complex queries)
4. **Background Work**: WorkManager with PeriodicWorkRequest (battery-optimized)
5. **API Integration**: Context7 as configured Retrofit client
6. **Secret Detection**: git-secrets with custom patterns
7. **Sparkline**: Canvas API (deferred to v2), daily snapshots (last 7 days)
8. **Feature Gating**: BuildConfig flags + SharedPreferences toggles

### Clarifications Resolved

All open questions have been resolved through research and best practices:

- ✅ **Context7 API Base URL**: Provisional `https://api.trading212.com/api/v0/` (configurable via BuildConfig)
- ✅ **Trading 212 API Rate Limits**: Provisional 60 req/min, 10,000/day with exponential backoff
- ✅ **Token Refresh Mechanism**: Long-lived API keys with manual renewal on 401 errors
- ✅ **Design System**: Material Design 3 + Android widget guidelines (no Figma dependency)
- ✅ **Minimum Android Version**: API 26 (Android 8.0) confirmed - 90%+ device coverage
- ✅ **Sparkline Granularity**: Daily data points (last 7 days) for v2 widget

See detailed analysis documents:
- [context7-api-analysis.md](./context7-api-analysis.md)
- [android-api-analysis.md](./android-api-analysis.md)
- [sparkline-granularity-analysis.md](./sparkline-granularity-analysis.md)

---

## Phase 1: Design (COMPLETED)

**Status**: ✅ Complete
**Outputs**: 
- [data-model.md](./data-model.md)
- [contracts/trading212-api.yaml](./contracts/trading212-api.yaml)
- [quickstart.md](./quickstart.md)
- [.github/copilot-instructions.md](/.github/copilot-instructions.md)

### Data Model:
- **Entities**: PortfolioSnapshot, Position, UserSettings
- **DTOs**: PortfolioDto, PositionDto (API response models)
- **Room Schema**: PortfolioEntity, PositionEntity (with cascade delete)
- **Enums**: RefreshFrequency, ThemeMode, WidgetVersion

### API Contracts:
- **GET /portfolio**: Retrieve full portfolio snapshot
- **GET /portfolio/{accountId}**: Account-specific snapshot
- **GET /positions**: List open positions with filters
- **GET /health**: Health check endpoint

### Architecture:
- Clean architecture: Presentation → Domain → Data layers
- Hilt dependency injection
- Repository pattern (remote + local data sources)
- WorkManager for background updates

---

## Phase 2: Constitution Re-evaluation

**Status**: ✅ Complete

All constitutional principles remain satisfied after design phase:

- [x] **Code Quality**: Architecture enforces modularity (presentation/domain/data separation), KDoc planned for public APIs, dependency versions will be pinned in gradle/libs.versions.toml
- [x] **Testing Standards**: Test structure planned (unit tests for repositories/use cases, Robolectric for ViewModels, Espresso for critical paths), coverage target maintained at ≥60%
- [x] **Security**: No secrets in design documents, EncryptedSharedPreferences for tokens, Room for cache (no plaintext storage), git-secrets installation documented in quickstart
- [x] **UX Consistency**: Widget sizes (1x1, 2x2, 4x1, 4x2) defined in layouts/, theme-aware resources planned (values/ + values-night/), accessibility standards (sp units, 48dp touch targets) documented
- [x] **Performance**: WorkManager intervals defined (15/30/60 min), cache-first strategy (optimistic display), Room query optimization (embedded P/L fields, indexed foreign keys), memory constraints documented (<50MB widget, <100MB app)
- [x] **Privacy**: Data flow is on-device only (API → Room → Widget), no off-device transmission, opt-in telemetry flag in UserSettings, privacy policy placeholder in UI
- [x] **CI/CD**: GitHub Actions workflows documented, ktlint/detekt integration planned, Dependabot enabled, signed release pipeline documented
- [x] **Release Criteria**: Quickstart includes testing commands, changelog process documented (GitHub Releases), migration guide placeholder, security review checklist (pre-commit hook + manual scan)

**No design changes required.** Ready for Phase 3 (Task Breakdown via `/speckit.tasks`).

---

## Next Steps

1. **Generate Tasks**: Run `/speckit.tasks` to break down implementation into executable tasks organized by user story
2. **Set Up Infrastructure**: Create Gradle configuration, Hilt modules, Room schema, Retrofit client
3. **Implement v1 Widget**: Build MVP with balance + P/L display in 2x2 layout
4. **Add Testing**: Write unit tests for core modules, set up CI pipeline
5. **Iterate to v2/v3**: Add positions (v2), then notifications (v3) based on feature flags

**Branch**: `002-trading-portfolio-widget`  
**Spec**: [spec.md](./spec.md)  
**Plan**: This file  
**Research**: [research.md](./research.md)  
**Data Model**: [data-model.md](./data-model.md)  
**Contracts**: [contracts/trading212-api.yaml](./contracts/trading212-api.yaml)  
**Quickstart**: [quickstart.md](./quickstart.md)

