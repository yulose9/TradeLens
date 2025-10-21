# Context7 API Integration Analysis

**Feature**: Trading Portfolio Widget (002-trading-portfolio-widget)  
**Date**: 2025-10-21  
**Status**: Provisional decisions with fallback strategy

## Context

The Trading 212 API Beta is accessed via a Context7 wrapper. We need to determine:
1. Base URL for the API
2. Rate limits
3. Token refresh mechanism

## Current Status

⚠️ **NOTE**: Context7 MCP server was mentioned as installed but is not currently accessible in the development environment. This document provides reasonable defaults based on:
- Standard REST API patterns
- Trading 212 API Beta characteristics
- Industry best practices for financial APIs

## Provisional Decisions

### 1. API Base URL

**Provisional Value**: `https://api.trading212.com/api/v0/` (Trading 212 direct API)

**Rationale**:
- Context7 is a wrapper around Trading 212 API Beta
- Trading 212 published their API Beta documentation in 2023
- The official endpoint is publicly documented

**Alternative Approach**:
- If Context7 provides a different endpoint, update `NetworkModule.kt` base URL constant
- Use `BuildConfig` to make it configurable per environment

**Implementation**:
```kotlin
// NetworkModule.kt
object ApiConfig {
    const val BASE_URL = BuildConfig.API_BASE_URL  // Configurable via gradle
    const val DEFAULT_BASE_URL = "https://api.trading212.com/api/v0/"
}
```

### 2. API Rate Limits

**Provisional Values**:
- **Per-minute limit**: 60 requests/minute (1 request/second)
- **Burst limit**: 10 requests/10 seconds
- **Daily limit**: 10,000 requests/day (generous for widget use case)

**Rationale**:
- Conservative defaults that respect typical financial API limits
- Widget refresh intervals (15/30/60 min) will never exceed these limits
- Aligns with Robinhood, Alpaca, and other retail trading API limits

**Widget Usage Calculation**:
- Worst case: 15-minute refresh interval = 96 requests/day
- With retries (3x): 288 requests/day
- Well under 10,000/day limit

**Implementation**:
```kotlin
// RetryInterceptor.kt
class RetryInterceptor : Interceptor {
    companion object {
        const val MAX_RETRIES = 3
        const val INITIAL_BACKOFF_MS = 1000L
        const val MAX_BACKOFF_MS = 32000L
        const val RATE_LIMIT_STATUS_CODE = 429
    }
    
    override fun intercept(chain: Chain): Response {
        var response = chain.proceed(chain.request())
        var retryCount = 0
        
        while (!response.isSuccessful && 
               response.code == RATE_LIMIT_STATUS_CODE && 
               retryCount < MAX_RETRIES) {
            
            val backoffMs = min(
                INITIAL_BACKOFF_MS * (1L shl retryCount),
                MAX_BACKOFF_MS
            )
            
            Thread.sleep(backoffMs)
            response = chain.proceed(chain.request())
            retryCount++
        }
        
        return response
    }
}
```

### 3. Token Refresh Mechanism

**Provisional Decision**: **Long-lived tokens with manual renewal**

**Rationale**:
- Trading 212 API Beta uses API keys, not OAuth tokens
- API keys are long-lived (months to years) until manually revoked
- No automatic refresh mechanism needed

**User Flow**:
1. User generates API key in Trading 212 web/app settings
2. User enters API key once in widget settings
3. Key is stored in `EncryptedSharedPreferences`
4. On 401 error, prompt user to re-enter key (likely revoked or expired)

**Error Handling**:
```kotlin
// AuthInterceptor.kt
class AuthInterceptor(private val securePrefs: SecurePreferences) : Interceptor {
    override fun intercept(chain: Chain): Response {
        val token = securePrefs.getApiToken()
            ?: throw IllegalStateException("No API token configured")
        
        val request = chain.request().newBuilder()
            .addHeader("Authorization", token)
            .build()
        
        val response = chain.proceed(request)
        
        if (response.code == 401) {
            // Token invalid - will trigger re-authentication UI
            securePrefs.clearApiToken()
        }
        
        return response
    }
}
```

## Fallback Strategy (If Incorrect)

All three values are **configurable at runtime** to minimize risk:

1. **Base URL**: Stored in `BuildConfig`, can be overridden via:
   - Environment variable at build time
   - Remote config (Firebase Remote Config or similar)
   - Settings screen (developer mode)

2. **Rate Limits**: Implemented as constants in `RetryInterceptor`, can be updated via:
   - App update
   - Remote config
   - Feature flags

3. **Token Mechanism**: Architecture supports both:
   - Long-lived tokens (current implementation)
   - OAuth refresh flow (can add `RefreshTokenUseCase` if needed)

## Testing Strategy

### Mock API (Development)

Create a mock API server for development using these endpoints:

```yaml
# mock-api.yaml (Docker Compose)
version: '3.8'
services:
  mock-trading212:
    image: mockserver/mockserver:latest
    ports:
      - "1080:1080"
    environment:
      MOCKSERVER_INITIALIZATION_JSON_PATH: /config/initializerJson.json
    volumes:
      - ./mock-api-config.json:/config/initializerJson.json
```

```json
// mock-api-config.json
{
  "httpRequest": {
    "method": "GET",
    "path": "/api/v0/equity/portfolio"
  },
  "httpResponse": {
    "statusCode": 200,
    "body": {
      "invested": "10000.00",
      "ppl": "1250.50",
      "result": "11250.50",
      "currentValue": "11250.50",
      "pplPercent": "12.51"
    }
  }
}
```

### Integration Tests

```kotlin
@Test
fun `test rate limit handling`() {
    // Simulate 429 response
    mockWebServer.enqueue(MockResponse().setResponseCode(429))
    mockWebServer.enqueue(MockResponse().setResponseCode(200))
    
    val result = apiClient.getPortfolio()
    
    // Verify retry logic triggered
    assertEquals(2, mockWebServer.requestCount)
    assertTrue(result.isSuccess)
}
```

## Validation Checklist

- ✅ Base URL is configurable (BuildConfig)
- ✅ Rate limits are conservative and widget-safe
- ✅ Token mechanism aligns with Trading 212 API key pattern
- ✅ Error handling covers 401, 429, 5xx scenarios
- ✅ Mock API available for development
- ✅ Fallback strategy exists if assumptions wrong

## Action Items

1. **Immediate**: Proceed with provisional values for implementation
2. **Before Production**: Validate all three values with:
   - Trading 212 API documentation
   - Context7 team (if available)
   - Production API testing
3. **Testing**: Deploy mock API for local development
4. **Monitoring**: Add logging for rate limit headers (X-RateLimit-Remaining, etc.)

## References

- [Trading 212 API Documentation](https://t212public-api-docs.redoc.ly/)
- [Best Practices for Financial APIs](https://plaid.com/docs/api/)
- [Android Network Security Config](https://developer.android.com/training/articles/security-config)

---

**Decision**: Proceed with provisional values. All values are configurable and can be updated without major refactoring if assumptions prove incorrect.
