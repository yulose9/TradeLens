# Android Minimum API Version Analysis

**Feature**: Trading Portfolio Widget (002-trading-portfolio-widget)  
**Date**: 2025-10-21  
**Status**: Decision - API 26 (Android 8.0 Oreo) confirmed

## Requirements Analysis

### Technical Requirements

1. **EncryptedSharedPreferences**: Requires API 23+ (Android 6.0 Marshmallow)
2. **Glance API (Jetpack Compose for Widgets)**: Works on API 21+, but optimal performance on API 31+ (Android 12)
3. **WorkManager**: Requires API 14+
4. **Retrofit + OkHttp**: Works on API 21+
5. **Room Database**: Requires API 16+

### Device Coverage Statistics (as of Q4 2024)

Based on Android Developer Dashboard and market research:

- **API 21 (Android 5.0)**: ~95%+ device coverage
- **API 23 (Android 6.0)**: ~94%+ device coverage
- **API 26 (Android 8.0)**: ~90%+ device coverage
- **API 28 (Android 9.0)**: ~85%+ device coverage
- **API 31 (Android 12)**: ~60%+ device coverage
- **API 33 (Android 13)**: ~40%+ device coverage

## Decision: API 26 (Android 8.0 Oreo)

### Rationale

1. **Market Coverage**: 90%+ device coverage is excellent for a niche financial widget
2. **Feature Support**: All required features work reliably on API 26+
3. **EncryptedSharedPreferences**: Fully supported (API 23+)
4. **Glance Widgets**: While optimal on API 31+, Glance provides compatibility layers for API 21+
5. **Security**: API 26+ includes important security improvements (TLS 1.2 default, stricter SSL)
6. **Maintenance**: Avoids legacy compatibility code for API 21-25

### Trade-offs Accepted

- **Glance Performance**: Some advanced widget features may have slightly degraded performance on API 26-30 vs API 31+
- **Device Exclusion**: Excludes ~10% of Android devices (mostly very old phones from 2016-2017)

### Alternative Considered

- **API 31 (Android 12)**: Would give optimal Glance performance but excludes 40% of devices - not acceptable for MVP

## Implementation Notes

### `app/build.gradle.kts`

```kotlin
android {
    compileSdk = 34
    
    defaultConfig {
        applicationId = "com.tradelens"
        minSdk = 26  // Android 8.0 Oreo
        targetSdk = 34  // Latest stable
        
        // Version info
        versionCode = 1
        versionName = "0.1.0"
    }
}
```

### Compatibility Considerations

1. **No API 21-25 Workarounds**: Clean codebase without legacy support code
2. **EncryptedSharedPreferences**: Use directly without fallback
3. **Glance Widgets**: May need to test rendering on API 26-30 for edge cases
4. **TLS/SSL**: Modern security standards work out of the box

## Testing Strategy

- **Emulators**: Test on API 26, 28, 31, 33, 34
- **Physical Devices**: If available, test on Android 8.0, 9.0, 12.0, 13.0, 14.0
- **Focus Areas**: Widget rendering across API levels, EncryptedSharedPreferences compatibility

## Validation

- ✅ Meets constitution security requirements (EncryptedSharedPreferences)
- ✅ Supports all planned features (Glance, WorkManager, Room, Retrofit)
- ✅ Covers 90%+ of target market
- ✅ No legacy compatibility code required
- ✅ Modern Android development practices

## References

- [Android Developer Dashboard](https://developer.android.com/about/dashboards)
- [EncryptedSharedPreferences Documentation](https://developer.android.com/reference/androidx/security/crypto/EncryptedSharedPreferences)
- [Glance API Compatibility](https://developer.android.com/jetpack/androidx/releases/glance)
- [Android Platform Versions](https://developer.android.com/about/versions)

---

**Decision**: Proceed with `minSdk = 26` as specified in plan.md and research.md. No changes needed.
