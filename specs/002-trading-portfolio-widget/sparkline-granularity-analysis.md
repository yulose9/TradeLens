# Sparkline Data Granularity Analysis

**Feature**: Trading Portfolio Widget (002-trading-portfolio-widget)  
**Date**: 2025-10-21  
**Status**: Decision - Daily data points (last 7 days)

## Context

The v2 widget (4x2 layout) will include a sparkline chart showing portfolio value trends over time. We need to determine the appropriate data granularity for this visualization.

## Options Considered

### Option 1: Hourly Data Points (Last 24 Hours)
- **Pros**: High-resolution data, shows intraday market movements
- **Cons**: 
  - Very granular for a widget (hard to read on small screen)
  - Requires frequent API calls or heavy caching
  - Trading 212 markets are closed on weekends (gaps in data)
  - Not useful for long-term trend visibility

### Option 2: Daily Data Points (Last 7 Days) ✅ SELECTED
- **Pros**:
  - Clean visualization on small widget canvas
  - Shows weekly trend (5 business days + 2 weekend days)
  - Aligns with common portfolio review patterns (weekly check-ins)
  - Reasonable cache size (7 snapshots stored in Room)
  - Minimal API load (1 update per day + user-triggered refreshes)
- **Cons**:
  - Doesn't show intraday volatility
  - May not capture sudden market events within a day

### Option 3: Snapshot-Based (Every Widget Update)
- **Pros**: Perfect accuracy to actual refresh frequency
- **Cons**:
  - Irregular time intervals (15/30/60 min depending on user settings)
  - Difficult to render meaningful trend (uneven x-axis)
  - Cache size unpredictable (could be 96+ snapshots per day)

## Decision: Daily Data Points (Last 7 Days)

### Rationale

1. **Widget Form Factor**: 4x2 widget has limited canvas space (~120x60dp usable area for sparkline). Daily granularity provides clear, readable visualization.

2. **User Intent**: Portfolio widgets are typically for "at-a-glance" monitoring, not active trading. Weekly trends are more actionable than hourly noise.

3. **Market Hours**: Trading 212 markets operate during business hours. Daily snapshots naturally align with market close times.

4. **Performance**: Storing 7 daily snapshots is lightweight (~2-3 KB per snapshot = ~20 KB total), meeting constitution performance requirements.

5. **API Efficiency**: Reduces unnecessary API calls. One "end of day" snapshot is sufficient for trend visualization.

6. **Industry Standard**: Most portfolio tracking apps (Robinhood, E*TRADE widgets) use daily granularity for sparklines.

## Implementation Details

### Data Storage Strategy

```kotlin
// PortfolioDao.kt
@Query("SELECT * FROM portfolio_snapshots ORDER BY timestamp DESC LIMIT 7")
suspend fun getLastSevenDays(): List<PortfolioSnapshot>

@Query("DELETE FROM portfolio_snapshots WHERE timestamp < :cutoffDate")
suspend fun deleteOlderThan(cutoffDate: Long)
```

### Cache Eviction Policy

- **Retention**: Keep last 7 days of daily snapshots
- **Eviction**: Delete snapshots older than 7 days during each sync
- **Snapshot Timing**: Capture one snapshot per day at first widget update after 00:00 UTC

### Sparkline Rendering

- **X-axis**: 7 data points evenly spaced (days)
- **Y-axis**: Portfolio value (auto-scale to min/max in dataset)
- **Visualization**: Smooth bezier curve connecting points
- **Labels**: Not displayed (widget too small), rely on visual trend only

### Room Schema Update

```kotlin
@Entity(tableName = "portfolio_snapshots")
data class PortfolioSnapshot(
    @PrimaryKey val timestamp: Long, // Unix timestamp (ms)
    val totalValue: BigDecimal,
    val totalPnL: BigDecimal,
    val pnlPercentage: Double,
    val dayOfYear: Int // For daily uniqueness (e.g., 365)
)

// Index for fast daily queries
@Index(value = ["dayOfYear"], unique = true)
```

### WorkManager Integration

```kotlin
// Sparkline sync job (runs once per day)
class DailySnapshotWorker : CoroutineWorker(context, params) {
    override suspend fun doWork(): Result {
        val today = LocalDate.now().dayOfYear
        
        // Check if we already have today's snapshot
        val existing = portfolioDao.getSnapshotForDay(today)
        if (existing != null) return Result.success()
        
        // Fetch fresh portfolio data
        val portfolio = apiRepository.getPortfolio()
        
        // Save snapshot
        portfolioDao.insert(portfolio.toSnapshot(dayOfYear = today))
        
        // Clean up old snapshots (>7 days)
        val cutoff = System.currentTimeMillis() - (7 * 24 * 60 * 60 * 1000)
        portfolioDao.deleteOlderThan(cutoff)
        
        return Result.success()
    }
}
```

## Testing Considerations

1. **Unit Tests**: Verify cache eviction logic (7-day retention)
2. **UI Tests**: Verify sparkline renders correctly with 1-7 data points
3. **Edge Cases**:
   - Fresh install (no historical data) - show flat line or "Insufficient data"
   - Single data point - show dot, not line
   - Market holidays - gaps in data handled gracefully

## Future Enhancements (Out of Scope for MVP)

- **v3**: Allow user to toggle between 7-day and 30-day views
- **v3**: Add hourly granularity as opt-in for active traders (feature flag)
- **v3**: Implement smart snapshot timing (market close detection)

## Validation

- ✅ Aligns with widget form factor constraints
- ✅ Meets performance requirements (<100ms rendering)
- ✅ Reasonable API load (minimal impact on rate limits)
- ✅ Industry best practice (7-day portfolio trend)
- ✅ Clean implementation (7 Room entries, simple query)

## References

- [Android Canvas API for Sparklines](https://developer.android.com/reference/android/graphics/Canvas)
- [Material Design Data Visualization](https://m3.material.io/foundations/content-design/data-visualization)
- [Robinhood Widget Design Patterns](https://newsroom.aboutrobinhood.com/)

---

**Decision**: Implement daily snapshots (last 7 days) for v2 widget sparkline feature. Update research.md to remove open question.
