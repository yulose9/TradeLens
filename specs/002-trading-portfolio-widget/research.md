# Research: Trading Portfolio Widget

**Phase**: 0 (Outline & Research)
**Date**: 2025-10-21
**Feature**: [spec.md](./spec.md) | [plan.md](./plan.md)

## Research Questions Resolved

### 1. Glance API for Widget Rendering

**Decision**: Use Jetpack Glance API for all widget layouts

**Rationale**: 
- Glance provides a declarative Compose-like API for app widgets, reducing boilerplate compared to RemoteViews
- Better integration with Jetpack Compose used for in-app settings screens
- Type-safe, composable widgets with automatic size adaptation
- Officially supported by Google as the modern approach for Android widgets
- Supports custom layouts for different widget sizes (1x1, 2x2, 4x1, 4x2)

**Alternatives considered**:
- RemoteViews (legacy approach): More verbose, error-prone, no Compose integration
- Hybrid (RemoteViews + Compose for settings): Inconsistent developer experience

**Implementation notes**:
- Use `GlanceAppWidget` as base class for `PortfolioWidget`
- Define size-specific composables: `CompactLayout`, `StandardLayout`, `WideLayout`, `ExpandedLayout`
- Use `LocalSize` to conditionally render layouts based on widget dimensions
- Glance requires Android 12+ for optimal experience, but degrades gracefully on API 26+

### 2. JSON Serialization Library Choice

**Decision**: Use Kotlinx.serialization for JSON parsing

**Rationale**:
- Kotlin-first design with compiler plugin for zero-reflection serialization
- Better performance than Moshi on Android (no reflection overhead)
- Native support for Kotlin features (default parameters, sealed classes, inline classes)
- Tight integration with Retrofit via `kotlinx-serialization-converter`
- Smaller binary size compared to Gson/Moshi

**Alternatives considered**:
- Moshi: Popular, well-tested, but requires reflection or codegen adapters (more setup)
- Gson: Legacy, reflection-heavy, poor Kotlin support (no default parameters)

**Implementation notes**:
- Add `kotlinx-serialization-json` dependency
- Configure Retrofit with `KotlinxSerializationConverterFactory`
- Use `@Serializable` annotation on DTOs
- Define custom serializers for BigDecimal (financial precision)

### 3. Room vs DataStore for Cache Storage

**Decision**: Use Room for portfolio snapshot cache

**Rationale**:
- Structured data with relationships (PortfolioSnapshot → List<Position>)
- Need to query historical snapshots for sparkline (v2 feature)
- Type-safe SQL queries with compile-time verification
- Better performance for complex queries compared to DataStore Proto
- Easier migration path if schema evolves

**Alternatives considered**:
- DataStore Proto: Good for simple key-value, but less suitable for relational data
- DataStore Preferences: No type safety, poor for structured data

**Implementation notes**:
- Use Room 2.5+ with Kotlin coroutines support
- Define `PortfolioEntity` and `PositionEntity` with one-to-many relationship
- Add `@Embedded` for denormalized P/L data (optimize read performance)
- Use `@TypeConverter` for timestamp and BigDecimal serialization
- Implement LRU cache eviction strategy (keep last 7 days of snapshots for sparkline)

### 4. WorkManager Configuration for Widget Updates

**Decision**: Use PeriodicWorkRequest with flexible intervals and exponential backoff

**Rationale**:
- WorkManager respects Android battery optimization constraints (Doze, App Standby)
- Guaranteed execution when constraints are met (network availability)
- Exponential backoff on API failures reduces battery drain and rate-limit violations
- Supports user-configurable refresh intervals via `PeriodicWorkRequest.Builder`
- Can trigger one-off updates for manual refresh via `OneTimeWorkRequest`

**Alternatives considered**:
- AlarmManager: Deprecated for periodic background work, poor battery optimization
- JobScheduler: Lower-level API, WorkManager is built on top and provides better ergonomics
- Foreground Service: Not suitable for widgets (user-initiated actions only)

**Implementation notes**:
- Define `WidgetUpdateWorker` extending `CoroutineWorker`
- Set constraints: `NetworkType.CONNECTED`, `BatteryNotLow` for battery-saver mode
- Map user refresh frequency to WorkManager intervals:
  - Fast: 15 minutes (minimum allowed by Android for PeriodicWorkRequest)
  - Normal: 30 minutes
  - Battery-saver: 60 minutes
- Note: Sub-15-minute "almost real-time" updates require WorkManager hacks (enqueue repeated one-off jobs) or foreground service (not recommended for widgets)
- Use `setBackoffCriteria` with `BackoffPolicy.EXPONENTIAL` for retry logic

### 5. Context7 API Integration Strategy

**Decision**: Implement Context7 as a configured Retrofit client with custom interceptors

**Rationale**:
- Context7 acts as a proxy/wrapper to Trading 212 API Beta
- Can leverage existing Retrofit infrastructure with Context7-specific configuration
- Interceptors handle Context7-specific headers, authentication, and error mapping
- Allows mocking Context7 responses in tests without external dependencies

**Alternatives considered**:
- Context7 SDK (if available): May introduce unnecessary dependencies and limit flexibility
- Direct Trading 212 API integration: Context7 may provide additional features (rate-limit handling, caching)

**Implementation notes**:
- Create `Context7ApiClient` class that configures `Retrofit.Builder` with:
  - Base URL: `https://api.trading212.com/api/v0/` (provisional, configurable via BuildConfig)
  - `AuthInterceptor`: Adds Trading 212 API key to Authorization header
  - `RetryInterceptor`: Handles 429 rate-limit responses with exponential backoff (60 req/min limit)
  - `LoggingInterceptor`: Logs requests/responses in debug builds only
- Define `Trading212Api` Retrofit interface with suspend functions for portfolio/positions endpoints
- Map Context7 error responses to domain-specific exceptions (`RateLimitException`, `AuthException`, `NetworkException`)

### 6. Pre-commit Hook Implementation

**Decision**: Use git-secrets or custom script with regex patterns

**Rationale**:
- git-secrets is AWS-maintained, well-tested, supports custom patterns
- Blocks commits containing secrets before they enter Git history
- Can scan entire history to detect existing secrets
- Low overhead, runs locally before commit

**Alternatives considered**:
- truffleHog: More advanced (entropy detection), but slower and may have false positives
- detect-secrets: Python-based, requires Python runtime in dev environment
- Custom script: Flexible but requires maintenance

**Implementation notes**:
- Install git-secrets via package manager (apt-get, brew, or manual)
- Configure patterns in `.git-hooks/pre-commit`:
  - `(api[-_]?key|token|secret|password)\s*[:=]\s*['"][^'"]{20,}['"]` (generic secrets)
  - Context7/Trading 212 specific token patterns (if known)
- Add to project setup documentation in README
- Automate installation in CI (scan all commits in PR)

### 7. Sparkline Rendering for 4x2 Widget

**Decision**: Use Canvas API with custom drawing logic, defer to v2

**Rationale**:
- Glance does not natively support Charts/Graphs (as of 2024)
- Canvas API allows custom drawing for sparkline (simple line chart)
- Keep sparkline minimal (last 7 days, single line, no axes/labels)
- Render as Bitmap, cache in memory, update on snapshot change

**Alternatives considered**:
- MPAndroidChart: Heavy library, not Glance-compatible
- Custom Compose Canvas: Works in-app but Glance requires RemoteViews-compatible rendering

**Implementation notes**:
- Defer sparkline implementation to v2 (v1 focuses on balance + P/L only)
- In v2, implement `SparklineRenderer` that:
  - Fetches last 7 snapshots from Room (order by timestamp)
  - Normalizes P/L values to 0-100 scale
  - Draws polyline on Canvas with theme-aware stroke color
  - Converts Canvas to Bitmap, cache in-memory (LRU cache)
  - Update Glance Image composable with bitmap

### 8. Feature-Gating Strategy

**Decision**: Use BuildConfig boolean flags + runtime SharedPreferences toggles

**Rationale**:
- BuildConfig flags allow compile-time gating for incomplete features (v2, v3)
- SharedPreferences toggles enable user opt-in for experimental features
- Supports gradual rollout and A/B testing (if remote config added later)

**Alternatives considered**:
- Remote config (Firebase Remote Config): Adds external dependency, requires network
- Build variants: More complex build configuration, harder to test multiple variants

**Implementation notes**:
- Define feature flags in `build.gradle.kts`:
  ```kotlin
  buildConfigField("Boolean", "FEATURE_V2_POSITIONS", "false")
  buildConfigField("Boolean", "FEATURE_V3_NOTIFICATIONS", "false")
  ```
- Store user preferences in SharedPreferences: `feature_v2_enabled`, `feature_v3_enabled`
- Gate widget layouts and settings UI based on flags:
  ```kotlin
  if (BuildConfig.FEATURE_V2_POSITIONS && userSettings.featureV2Enabled) {
      // Show positions in widget
  }
  ```

## Technology Stack Summary

| Category | Technology | Version | Rationale |
|----------|-----------|---------|-----------|
| Language | Kotlin | 1.9+ | Modern, null-safe, concise, official Android language |
| UI Framework | Jetpack Compose | 1.5+ | Declarative UI, official modern toolkit |
| Widget API | Glance | 1.0+ | Compose-like API for widgets |
| Networking | Retrofit + OkHttp | 2.9+ / 4.x | Industry standard, type-safe, interceptor support |
| JSON | Kotlinx.serialization | 1.6+ | Kotlin-first, zero-reflection, performance |
| Background | WorkManager | 2.8+ | Battery-optimized, guaranteed execution |
| Cache | Room | 2.5+ | Type-safe SQL, relational data support |
| Secrets | EncryptedSharedPreferences | 1.1+ | Secure storage for tokens |
| DI | Hilt | 2.48+ | Android-optimized Dagger, less boilerplate |
| Testing | JUnit + MockK | 4.13+ / 1.13+ | Standard unit testing stack |
| Static Analysis | ktlint + detekt | 0.50+ / 1.23+ | Code style + complexity checks |
| CI/CD | GitHub Actions | N/A | Free for public repos, tight integration |

## Open Questions / RESOLVED

All clarifications have been resolved through research and best practices analysis:

1. **Context7 Endpoint**: ✅ **RESOLVED** - Provisional: `https://api.trading212.com/api/v0/` (Trading 212 direct API). Configurable via BuildConfig. See [context7-api-analysis.md](./context7-api-analysis.md).
2. **Trading 212 API Rate Limits**: ✅ **RESOLVED** - Provisional: 60 req/min, 10,000/day. Conservative defaults with exponential backoff. See [context7-api-analysis.md](./context7-api-analysis.md).
3. **Design System**: ✅ **RESOLVED** - Material Design 3 and Android widget design guidelines will be followed for colors, typography, spacing, and iconography to ensure consistency and accessibility. See spec.md UX-001 for references.
4. **Minimum Android Version**: ✅ **RESOLVED** - API 26 (Android 8.0) confirmed, provides 90%+ device coverage. See [android-api-analysis.md](./android-api-analysis.md).
5. **Token Refresh Mechanism**: ✅ **RESOLVED** - Long-lived API keys with manual renewal (standard for Trading 212 API Beta). 401 errors trigger re-authentication UI. See [context7-api-analysis.md](./context7-api-analysis.md).
6. **Sparkline Data Granularity**: ✅ **RESOLVED** - Daily snapshots (last 7 days) selected for optimal widget visualization. See [sparkline-granularity-analysis.md](./sparkline-granularity-analysis.md).

## Next Steps

1. **Phase 1**: Generate data-model.md (define entities and relationships)
2. **Phase 1**: Generate API contracts (OpenAPI spec for Trading 212/Context7 endpoints)
3. **Phase 1**: Generate quickstart.md (developer setup guide)
4. **Phase 1**: Update agent context file with technology stack
5. **Phase 2**: Re-evaluate Constitution Check post-design
6. **Phase 2**: Generate tasks.md (implementation task breakdown)
