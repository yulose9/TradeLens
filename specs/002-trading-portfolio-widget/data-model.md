# Data Model: Trading Portfolio Widget

**Phase**: 1 (Design & Contracts)
**Date**: 2025-10-21
**Feature**: [spec.md](./spec.md) | [plan.md](./plan.md) | [research.md](./research.md)

## Entity Overview

This document defines the core domain entities, their relationships, validation rules, and state transitions for the Trading Portfolio Widget application.

## Domain Entities

### 1. PortfolioSnapshot

Represents a point-in-time snapshot of the user's entire portfolio, including total value, profit/loss, and timestamp.

**Fields**:
- `id`: Long (primary key, auto-generated)
- `totalValue`: BigDecimal (total portfolio value in base currency)
- `currency`: String (ISO 4217 currency code, e.g., "USD", "EUR")
- `profitLossAbsolute`: BigDecimal (absolute P/L since inception)
- `profitLossPercentage`: BigDecimal (percentage P/L since inception, e.g., 15.5 for 15.5%)
- `timestamp`: Instant (UTC timestamp of snapshot)
- `positions`: List<Position> (one-to-many relationship)
- `isStale`: Boolean (computed: true if timestamp > refresh interval)

**Validation Rules**:
- `totalValue` >= 0
- `currency` must be valid ISO 4217 code (3 uppercase letters)
- `profitLossPercentage` = (profitLossAbsolute / (totalValue - profitLossAbsolute)) * 100
- `timestamp` <= current time (no future timestamps)
- `positions` can be empty (valid for new accounts)

**Relationships**:
- One PortfolioSnapshot → Many Positions (cascade delete)

**State Transitions**:
- Fresh → Stale: When current_time > (timestamp + refresh_interval)
- N/A → Loaded: When first snapshot retrieved from API
- Loaded → Updated: When new snapshot replaces cached snapshot

**Notes**:
- Store in Room database for offline access
- Keep last 7 days of snapshots for sparkline (v2 feature)
- Use `@Embedded` for denormalized P/L fields to optimize read performance

---

### 2. Position

Represents a single open position (stock/crypto holding) within a portfolio snapshot.

**Fields**:
- `id`: Long (primary key, auto-generated)
- `snapshotId`: Long (foreign key to PortfolioSnapshot)
- `symbol`: String (ticker symbol, e.g., "AAPL", "BTC")
- `name`: String (full name, e.g., "Apple Inc.", "Bitcoin")
- `quantity`: BigDecimal (number of shares/units held)
- `entryPrice`: BigDecimal (average purchase price per unit)
- `currentPrice`: BigDecimal (current market price per unit)
- `profitLossAbsolute`: BigDecimal (absolute P/L for this position)
- `profitLossPercentage`: BigDecimal (percentage P/L for this position)
- `positionValue`: BigDecimal (current value = quantity * currentPrice)

**Validation Rules**:
- `symbol` must be non-empty, alphanumeric, max 10 characters
- `quantity` > 0
- `entryPrice` > 0
- `currentPrice` >= 0 (can be 0 for halted/delisted assets)
- `profitLossAbsolute` = (currentPrice - entryPrice) * quantity
- `profitLossPercentage` = ((currentPrice - entryPrice) / entryPrice) * 100
- `positionValue` = quantity * currentPrice

**Relationships**:
- Many Positions → One PortfolioSnapshot

**Display Rules**:
- Sort by `positionValue` descending (show largest holdings first)
- Show top 5 positions in 2x2 widget
- Show top 10 positions in 4x2 widget
- Use color coding: green for positive P/L, red for negative P/L

**Notes**:
- Store as child entity in Room with `@Relation` to PortfolioSnapshot
- Cache last known price for offline display

---

### 3. UserSettings

Stores user preferences and configuration, including API credentials, refresh frequency, and widget display options.

**Fields**:
- `apiToken`: String (Trading 212 API token, encrypted)
- `refreshFrequency`: RefreshFrequency (enum: FAST, NORMAL, BATTERY_SAVER)
- `selectedAccountIds`: List<String> (filter to specific accounts, empty = all accounts)
- `selectedPositionSymbols`: List<String> (filter to specific positions, empty = all positions)
- `themeMode`: ThemeMode (enum: LIGHT, DARK, SYSTEM)
- `enableV2Features`: Boolean (opt-in for positions + sparkline)
- `enableV3Features`: Boolean (opt-in for notifications + thresholds)
- `lastSuccessfulSync`: Instant (timestamp of last successful API call)
- `enableTelemetry`: Boolean (opt-in for optional analytics)

**Validation Rules**:
- `apiToken` must be non-empty if set (stored in EncryptedSharedPreferences)
- `refreshFrequency` maps to WorkManager intervals:
  - FAST: 15 minutes (minimum allowed)
  - NORMAL: 30 minutes
  - BATTERY_SAVER: 60 minutes
- `selectedAccountIds` and `selectedPositionSymbols` can be empty (default: show all)
- `lastSuccessfulSync` can be null (never synced yet)

**Storage**:
- `apiToken`: EncryptedSharedPreferences (key: "api_token")
- Other fields: SharedPreferences or DataStore Preferences (key-value pairs)

**State Transitions**:
- Unauthenticated → Authenticated: When user enters valid API token
- Authenticated → Unauthenticated: When token is invalid/expired (401 response)
- Sync Pending → Synced: When API call succeeds and `lastSuccessfulSync` updates

**Notes**:
- Provide default values: `refreshFrequency = NORMAL`, `themeMode = SYSTEM`, features disabled
- Validate token on first use, prompt re-authentication if invalid

---

## Enumerations

### RefreshFrequency
```kotlin
enum class RefreshFrequency(val intervalMinutes: Long) {
    FAST(15),        // Almost real-time (WorkManager minimum)
    NORMAL(30),      // Balanced
    BATTERY_SAVER(60) // Battery-friendly
}
```

### ThemeMode
```kotlin
enum class ThemeMode {
    LIGHT,   // Force light theme
    DARK,    // Force dark theme
    SYSTEM   // Follow system theme
}
```

### WidgetVersion
```kotlin
enum class WidgetVersion {
    V1,  // Balance + P/L only
    V2,  // + Positions + Sparkline
    V3   // + Notifications + Thresholds
}
```

---

## DTOs (Data Transfer Objects)

DTOs represent API response models from Trading 212/Context7. These are separate from domain entities to decouple API contract from internal data model.

### PortfolioDto
```kotlin
@Serializable
data class PortfolioDto(
    val totalValue: String,        // API returns as string to preserve precision
    val currency: String,
    val profitLoss: String,
    val profitLossPercentage: String,
    val positions: List<PositionDto>
)
```

### PositionDto
```kotlin
@Serializable
data class PositionDto(
    val symbol: String,
    val name: String,
    val quantity: String,
    val averagePrice: String,
    val currentPrice: String,
    val profitLoss: String,
    val profitLossPercentage: String
)
```

**Mapping Notes**:
- Convert string decimals to `BigDecimal` for precision (use `toBigDecimal()`)
- Generate `timestamp` on client side when DTO is received
- Compute `positionValue` from quantity and currentPrice

---

## Room Database Schema

### PortfolioEntity (Table: portfolio_snapshots)
```kotlin
@Entity(tableName = "portfolio_snapshots")
data class PortfolioEntity(
    @PrimaryKey(autoGenerate = true) val id: Long = 0,
    val totalValue: String,              // Store as string to preserve BigDecimal precision
    val currency: String,
    val profitLossAbsolute: String,
    val profitLossPercentage: String,
    val timestamp: Long                  // Unix epoch millis
)
```

### PositionEntity (Table: positions)
```kotlin
@Entity(
    tableName = "positions",
    foreignKeys = [ForeignKey(
        entity = PortfolioEntity::class,
        parentColumns = ["id"],
        childColumns = ["snapshotId"],
        onDelete = ForeignKey.CASCADE
    )],
    indices = [Index("snapshotId")]
)
data class PositionEntity(
    @PrimaryKey(autoGenerate = true) val id: Long = 0,
    val snapshotId: Long,
    val symbol: String,
    val name: String,
    val quantity: String,
    val entryPrice: String,
    val currentPrice: String,
    val profitLossAbsolute: String,
    val profitLossPercentage: String
)
```

### Relationships
```kotlin
data class PortfolioWithPositions(
    @Embedded val portfolio: PortfolioEntity,
    @Relation(
        parentColumn = "id",
        entityColumn = "snapshotId"
    )
    val positions: List<PositionEntity>
)
```

---

## Data Flow

1. **API Response → DTO**: Retrofit deserializes JSON to `PortfolioDto` using Kotlinx.serialization
2. **DTO → Domain Entity**: Mapper converts `PortfolioDto` to `PortfolioSnapshot`, applying validation and computing derived fields
3. **Domain Entity → Room Entity**: Mapper converts `PortfolioSnapshot` to `PortfolioEntity` for persistence
4. **Room Entity → Domain Entity**: Mapper converts `PortfolioEntity` to `PortfolioSnapshot` for use in ViewModels/UseCases
5. **Domain Entity → UI State**: ViewModels transform `PortfolioSnapshot` to display-specific state for Glance widgets and Compose screens

**Key Mappers**:
- `PortfolioDtoMapper.toDomain(dto: PortfolioDto): PortfolioSnapshot`
- `PortfolioEntityMapper.toEntity(domain: PortfolioSnapshot): PortfolioEntity`
- `PortfolioEntityMapper.toDomain(entity: PortfolioWithPositions): PortfolioSnapshot`

---

## Cache Eviction Strategy

To prevent unbounded database growth and maintain sparkline data:

1. **Retention Policy**: Keep last 7 days of snapshots (168 snapshots at 1-hour granularity)
2. **Eviction Trigger**: Run cleanup on every new snapshot insert
3. **Eviction Query**: `DELETE FROM portfolio_snapshots WHERE timestamp < (current_timestamp - 7 days)`
4. **Cascade Delete**: Positions are automatically deleted via Room `onDelete = CASCADE` foreign key constraint

---

## Error Handling

### API Error Mapping
- 401 Unauthorized → `AuthException("Invalid or expired API token")`
- 429 Too Many Requests → `RateLimitException("Rate limit exceeded, retry after X seconds")`
- 503 Service Unavailable → `NetworkException("Service unavailable, try again later")`
- Network timeout → `NetworkException("Network timeout")`

### Validation Errors
- Invalid BigDecimal conversion → Log error, use 0 as fallback
- Negative values → Log error, clamp to 0
- Missing required fields → Log error, skip entity (don't crash)

---

## Summary

This data model provides:
- **Clear separation**: DTOs (API) → Domain Entities (business logic) → Room Entities (persistence)
- **Type safety**: BigDecimal for financial precision, enums for states, nullable types for optional fields
- **Offline-first**: Room cache enables instant widget display without network
- **Performance**: Denormalized P/L fields, indexed foreign keys, LRU cache eviction
- **Scalability**: Supports 10-100 positions per snapshot, 7 days of historical data for sparkline
