# BuzzebeesSDK Comprehensive Guide

This document provides a complete overview of the BuzzebeesSDK, including all core components, custom API integration, and advanced features. This guide serves as both a reference and implementation guide for developers.

---

## Table of Contents

1. [SDK Architecture Overview](#sdk-architecture-overview)
2. [Core Components](#core-components)
3. [Custom API Integration](#custom-api-integration)
4. [URL Building Extensions](#url-building-extensions)
5. [Authentication & Token Management](#authentication--token-management)
6. [Badge Management System](#badge-management-system)
7. [Error Handling Framework](#error-handling-framework)
8. [Best Practices](#best-practices)
9. [Integration Examples](#integration-examples)
10. [Troubleshooting](#troubleshooting)

---

## SDK Architecture Overview

The BuzzebeesSDK follows a layered architecture design pattern:

```
┌─────────────────────────────────────┐
│           BuzzebeesSDK              │  ← Main SDK Entry Point
├─────────────────────────────────────┤
│         Use Case Layer              │  ← Business Logic (Auth, Campaign, etc.)
├─────────────────────────────────────┤
│        Service Manager              │  ← Service Coordination
├─────────────────────────────────────┤
│         API Client                  │  ← HTTP Configuration & API Services
├─────────────────────────────────────┤
│       Network Layer                 │  ← OkHttp, Retrofit, Interceptors
└─────────────────────────────────────┘
```

### Key Architectural Benefits

- **Separation of Concerns**: Each layer has a specific responsibility
- **Dependency Injection**: Dependencies flow downward through layers
- **Extensibility**: Custom APIs can leverage existing infrastructure
- **Testability**: Each layer can be tested independently
- **Maintainability**: Clear boundaries and responsibilities

---

## Core Components

### 1. BuzzebeesSDK (Main Entry Point)

The main SDK class provides access to all functionality:

```kotlin
// Initialize SDK (automatic via ContentProvider)
val sdk = BuzzebeesSDK.instance()

// Access core services
val authUseCase = sdk.auth
val campaignUseCase = sdk.campaign
val profileUseCase = sdk.profile
val badgeManager = sdk.badgeManager

// Access configuration and providers
val authProvider = sdk.authProvider()
val configs = sdk.servicesConfigs()
val deviceInfo = sdk.deviceInfoProvider()
```

### 2. Use Cases (Business Logic Layer)

Each use case encapsulates business logic for a specific domain:

| Use Case | Purpose | Key Methods |
|----------|---------|-------------|
| **AuthUseCase** | Authentication & login | `deviceLogin()`, `usernamePasswordLogin()`, `logout()` |
| **CampaignUseCase** | Campaign management | `getCampaigns()`, `getCampaignDetail()`, `redeemCampaign()` |
| **ProfileUseCase** | User profile management | `getProfile()`, `updateProfile()`, `uploadProfileImage()` |
| **BadgeUseCase** | Badge operations | `getBadges()`, `earnBadge()`, `getBadgeHistory()` |
| **WalletUseCase** | Wallet & transactions | `getWallet()`, `getTransactions()`, `topUp()` |

### 3. Service Manager

Coordinates all services and manages dependencies:

```kotlin
class ServiceManager(private val dependencies: SDKDependencies) {
    // Lazy initialization of all use cases
    val authUseCase: AuthUseCase by lazy { /* ... */ }
    val campaignUseCase: CampaignUseCase by lazy { /* ... */ }
    
    // Custom API support
    fun getRetrofit(): Retrofit = dependencies.apiClient.getRetrofit()
    fun getOkHttpClient(): OkHttpClient = dependencies.apiClient.getOkHttpClient()
}
```

### 4. API Client (HTTP Configuration)

Manages HTTP configuration and API service creation:

```kotlin
class ApiClient(private val retrofit: Retrofit) {
    // Built-in API services
    val authApi: AuthApi by lazy { retrofit.create(AuthApi::class.java) }
    val campaignApi: CampaignApi by lazy { retrofit.create(CampaignApi::class.java) }
    
    // Custom API support
    fun getRetrofit(): Retrofit = retrofit
    fun getOkHttpClient(): OkHttpClient = okHttpClient
}
```

---

## Custom API Integration

The SDK provides comprehensive custom API support while leveraging existing infrastructure:

### Simple Custom API Creation

```kotlin
// Define your API interface
interface WeatherApi {
    @GET("weather/current")
    suspend fun getCurrentWeather(@Query("city") city: String): WeatherResponse
}

// Create API service using SDK configuration
val weatherApi = BuzzebeesSDK.instance().createCustomApiService(WeatherApi::class.java)
```

### Advanced Custom API Configuration

```kotlin
val customApi = BuzzebeesSDK.instance().customApiBuilder()
    .baseUrl("https://api.example.com/v2/")
    .addInterceptor(ApiKeyInterceptor("your-api-key"))
    .addInterceptor(CustomLoggingInterceptor("CustomAPI"))
    .converterFactory(MoshiConverterFactory.create())
    .build(CustomApi::class.java)
```

### Direct HTTP Access

```kotlin
val okHttpClient = BuzzebeesSDK.instance().getConfiguredOkHttpClient()
val request = Request.Builder()
    .url("https://api.example.com/data")
    .build()

val response = okHttpClient.newCall(request).execute()
```

### Custom API Provider Interface

```kotlin
val provider = BuzzebeesSDK.instance().getCustomApiProvider()

// Create multiple APIs with same configuration
val weatherApi = provider.createApiService(WeatherApi::class.java)
val paymentApi = provider.createApiService(PaymentApi::class.java)

// Access underlying components
val retrofit = provider.getRetrofit()
val okHttpClient = provider.getOkHttpClient()
```

---

## URL Building Extensions

The SDK provides convenient URL building capabilities:

### Profile Image URLs

```kotlin
val profileImageUrl = BuzzebeesSDK.instance().buildProfileImageUrl(userId)
// Automatically includes authentication token and base URL
```

### Campaign Image URLs

```kotlin
val campaignImageUrl = BuzzebeesSDK.instance().buildFullImageUrl(
    image = campaign.imageUrl,
    id = campaign.id
)
```

### Website URLs

```kotlin
val websiteUrl = BuzzebeesSDK.instance().buildWebsiteUrl(campaign)
```

### Redemption URLs

```kotlin
val redeemOptions = RedeemOptions(
    color = "red",
    size = "L", 
    quantity = 2
)

val redeemUrl = BuzzebeesSDK.instance().buildRedeemUrl(
    campaign = campaign,
    userId = userId,
    options = redeemOptions
)
```

### Direct URL Builder Access

```kotlin
val urlBuilder = BuzzebeesSDK.instance().urlBuilder
// Access to BuzzebeesUrlBuilder for advanced URL construction
```

---

## Authentication & Token Management

The AuthProvider handles all authentication and token management:

### Token Types

| Token Type | Purpose | Storage Method |
|------------|---------|----------------|
| **Auth Token** | Primary authentication | `saveAuthToken()` / `getAuthToken()` |
| **JWT Token** | JSON Web Token | `saveJwtToken()` / `getJwtToken()` |
| **E-Wallet Token** | E-wallet operations | `saveEWalletToken()` / `getEWalletToken()` |
| **Wallet Token** | Wallet transactions | `saveWalletToken()` / `getWalletToken()` |
| **FCM Token** | Push notifications | `saveFcmToken()` / `getFcmToken()` |

### Locale Management

```kotlin
val authProvider = BuzzebeesSDK.instance().authProvider()

// Save user locale
authProvider.saveLocale(1054) // Thai
authProvider.saveLocale(1033) // English

// Retrieve locale
val locale = authProvider.getLocale()

// Check locale existence
if (authProvider.hasLocale()) {
    // Handle locale-specific logic
}
```

### Bulk Token Operations

```kotlin
// Update multiple tokens at once
authProvider.updateTokens(
    authToken = "new_auth_token",
    jwtToken = "new_jwt_token",
    locale = 1054
)

// Clear all tokens
authProvider.clear()

// Check token status
val tokenInfo = authProvider.getTokenInfo()
```

---

## Badge Management System

Real-time badge management with lifecycle-aware integration:

### Basic Badge Integration

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        val badgeManager = BuzzebeesSDK.instance().badgeManager
        
        // Observe badge updates with automatic lifecycle management
        this.observeBadgeUpdates(badgeManager) { badgeModel ->
            runOnUiThread {
                updateBadgeUI(badgeModel.count)
            }
        }
    }
}
```

### Advanced Badge Management

```kotlin
class BadgeViewModel : ViewModel() {
    private val badgeManager = BuzzebeesSDK.instance().badgeManager
    
    private val _badgeCount = MutableLiveData<Int>()
    val badgeCount: LiveData<Int> = _badgeCount
    
    private val badgeListener = object : BadgeListener {
        override fun onBadgeReceived(badgeModel: BadgeTraceModel) {
            _badgeCount.postValue(badgeModel.count)
        }
    }
    
    init {
        badgeManager.registerBadgeListener(badgeListener)
    }
    
    override fun onCleared() {
        super.onCleared()
        badgeManager.unregisterBadgeListener(badgeListener)
    }
}
```

### Direct Badge Data Access

```kotlin
lifecycleScope.launch {
    BuzzebeesSDK.instance().badgeManager.getBadgeModelFlow()
        .collect { badgeModel ->
            badgeModel?.let { 
                // Use badge data
                updateUI(it.appPoint, it.count)
            }
        }
}
```

---

## Error Handling Framework

Comprehensive error handling across all SDK operations:

### Centralized Error Handler

```kotlin
class SDKErrorHandler {
    fun handleError(context: Context, error: Throwable) {
        when (error) {
            is HttpException -> handleHttpError(context, error)
            is IOException -> handleNetworkError(context)
            is JsonSyntaxException -> handleParsingError(context)
            else -> handleUnknownError(context, error)
        }
    }
}
```

### Use Case Error Handling

```kotlin
class AuthService {
    private val errorHandler = SDKErrorHandler()
    
    suspend fun login(username: String, password: String): Result<LoginResponse> {
        return try {
            val response = BuzzebeesSDK.instance().auth.usernamePasswordLogin(username, password)
            Result.success(response)
        } catch (e: Exception) {
            errorHandler.handleError(context, e)
            Result.failure(e)
        }
    }
}
```

### Error Types and Handling

| Error Type | Common Causes | Handling Strategy |
|------------|---------------|-------------------|
| **HttpException** | API errors, invalid requests | Show specific error message, retry option |
| **IOException** | Network connectivity issues | Show network error, retry mechanism |
| **JsonSyntaxException** | API response parsing errors | Log error, show generic message |
| **AuthenticationException** | Invalid credentials, expired tokens | Redirect to login, clear tokens |
| **ValidationException** | Invalid input data | Show field-specific errors |

---

## Best Practices

### 1. SDK Initialization

```kotlin
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        // SDK initializes automatically via ContentProvider
        // No manual initialization needed
    }
}
```

### 2. Lifecycle Management

```kotlin
// Fragment with proper lifecycle management
class MyFragment : Fragment() {
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        // Use viewLifecycleOwner for proper cleanup
        val badgeManager = BuzzebeesSDK.instance().badgeManager
        viewLifecycleOwner.observeBadgeUpdates(badgeManager) { badgeModel ->
            updateBadgeDisplay(badgeModel.count)
        }
    }
}
```

### 3. Error Handling Strategy

```kotlin
class ApiService {
    suspend fun fetchData(): Result<DataResponse> {
        return try {
            val response = api.getData()
            Result.success(response)
        } catch (e: HttpException) {
            when (e.code()) {
                401 -> Result.failure(AuthenticationException("Authentication required"))
                403 -> Result.failure(AuthorizationException("Access denied"))
                404 -> Result.failure(NotFoundException("Resource not found"))
                500 -> Result.failure(ServerException("Server error"))
                else -> Result.failure(ApiException("API error: ${e.message}"))
            }
        } catch (e: IOException) {
            Result.failure(NetworkException("Network error: ${e.message}"))
        } catch (e: Exception) {
            Result.failure(UnknownException("Unknown error: ${e.message}"))
        }
    }
}
```

### 4. Resource Management

```kotlin
class ResourceManager {
    private val customApi = BuzzebeesSDK.instance().createCustomApiService(CustomApi::class.java)
    private val okHttpClient = BuzzebeesSDK.instance().getConfiguredOkHttpClient()
    
    // Reuse instances - don't create new ones frequently
    fun getCustomApi() = customApi
    fun getHttpClient() = okHttpClient
}
```

### 5. Dependency Injection Integration

```kotlin
@Module
class SDKModule {
    
    @Provides
    @Singleton
    fun provideBuzzebeesSDK(): BuzzebeesSDK = BuzzebeesSDK.instance()
    
    @Provides
    @Singleton
    fun provideAuthProvider(sdk: BuzzebeesSDK): AuthProvider = sdk.authProvider()
    
    @Provides
    @Singleton
    fun provideBadgeManager(sdk: BuzzebeesSDK): BadgeManager = sdk.badgeManager
    
    @Provides
    fun provideCustomApiProvider(sdk: BuzzebeesSDK): CustomApiProvider = 
        sdk.getCustomApiProvider()
}
```

---

## Integration Examples

### Complete Authentication Flow

```kotlin
class AuthenticationManager {
    private val sdk = BuzzebeesSDK.instance()
    private val authProvider = sdk.authProvider()
    
    suspend fun loginWithCredentials(username: String, password: String): AuthResult {
        return try {
            // Attempt login
            val response = sdk.auth.usernamePasswordLogin(username, password)
            
            // Save tokens
            authProvider.saveAuthToken(response.authToken)
            authProvider.saveJwtToken(response.jwtToken)
            authProvider.saveLocale(response.locale)
            
            AuthResult.Success(response.user)
        } catch (e: HttpException) {
            when (e.code()) {
                401 -> AuthResult.InvalidCredentials
                429 -> AuthResult.TooManyAttempts
                else -> AuthResult.Error(e.message ?: "Login failed")
            }
        } catch (e: Exception) {
            AuthResult.Error(e.message ?: "Unknown error")
        }
    }
    
    fun isUserLoggedIn(): Boolean {
        return authProvider.hasAuthToken()
    }
    
    fun logout() {
        lifecycleScope.launch {
            try {
                sdk.auth.logout()
            } catch (e: Exception) {
                // Log error but continue with local cleanup
                Log.e("Auth", "Logout API call failed", e)
            } finally {
                authProvider.clear()
            }
        }
    }
}
```

### Campaign Management Integration

```kotlin
class CampaignService {
    private val sdk = BuzzebeesSDK.instance()
    
    suspend fun loadCampaign(campaignId: String): CampaignDetail? {
        return try {
            val campaign = sdk.campaign.getCampaignDetail(campaignId)
            
            // Build URLs using SDK extensions
            val imageUrl = sdk.buildFullImageUrl(campaign.imageUrl, campaign.id)
            val websiteUrl = sdk.buildWebsiteUrl(campaign)
            
            CampaignDetail(
                campaign = campaign,
                imageUrl = imageUrl,
                websiteUrl = websiteUrl
            )
        } catch (e: Exception) {
            Log.e("Campaign", "Failed to load campaign: $campaignId", e)
            null
        }
    }
    
    suspend fun redeemCampaign(
        campaignId: String, 
        userId: String, 
        options: RedeemOptions
    ): RedeemResult {
        return try {
            val response = sdk.campaign.redeemCampaign(campaignId, options)
            RedeemResult.Success(response.redeemId)
        } catch (e: HttpException) {
            when (e.code()) {
                400 -> RedeemResult.InvalidRequest
                403 -> RedeemResult.NotEligible
                409 -> RedeemResult.AlreadyRedeemed
                429 -> RedeemResult.LimitExceeded
                else -> RedeemResult.Error(e.message ?: "Redemption failed")
            }
        } catch (e: Exception) {
            RedeemResult.Error(e.message ?: "Unknown error")
        }
    }
}
```

### Multi-API Integration

```kotlin
class IntegratedApiService {
    private val sdk = BuzzebeesSDK.instance()
    
    // Custom APIs using SDK configuration
    private val weatherApi = sdk.customApiBuilder()
        .baseUrl("https://weather-api.com/v2/")
        .addInterceptor(ApiKeyInterceptor(BuildConfig.WEATHER_API_KEY))
        .build(WeatherApi::class.java)
    
    private val paymentApi = sdk.customApiBuilder()
        .baseUrl("https://payment-gateway.com/api/")
        .addInterceptor(PaymentAuthInterceptor())
        .build(PaymentApi::class.java)
    
    suspend fun processWeatherBasedCampaign(cityId: String, userId: String): CampaignResult {
        return try {
            // Get weather data
            val weather = weatherApi.getCurrentWeather(cityId)
            
            // Get relevant campaigns based on weather
            val campaigns = sdk.campaign.getCampaigns(
                filters = CampaignFilters(weather = weather.condition)
            )
            
            val applicableCampaign = campaigns.firstOrNull { it.isApplicableForWeather(weather) }
                ?: return CampaignResult.NoCampaignsAvailable
            
            // Generate redemption URL
            val redeemUrl = sdk.buildRedeemUrl(
                campaign = applicableCampaign,
                userId = userId,
                options = RedeemOptions(weatherCondition = weather.condition)
            )
            
            CampaignResult.Success(applicableCampaign, redeemUrl)
        } catch (e: Exception) {
            CampaignResult.Error(e.message ?: "Failed to process campaign")
        }
    }
}
```

---

## Troubleshooting

### Common Issues and Solutions

#### 1. SDK Not Initialized

**Symptoms**: `IllegalStateException: ServiceSDK is not initialized`

**Solution**:
```kotlin
// Check if ContentProvider is properly configured in AndroidManifest.xml
<provider
    android:name="com.buzzebees.sdk.BuzzebeesSDK$LibProvider"
    android:authorities="${applicationId}.buzzebees.sdk"
    android:enabled="true"
    android:exported="false" />
```

#### 2. Authentication Issues

**Symptoms**: HTTP 401 errors, authentication failures

**Solutions**:
```kotlin
// Check token validity
val authProvider = BuzzebeesSDK.instance().authProvider()
if (!authProvider.hasAuthToken()) {
    // Redirect to login
    redirectToLogin()
    return
}

// Check token info for debugging
val tokenInfo = authProvider.getTokenInfo()
Log.d("Auth", "Token info: $tokenInfo")

// Clear tokens if corrupted
if (tokenInfo.authTokenLength == 0) {
    authProvider.clear()
    redirectToLogin()
}
```

#### 3. Network Configuration Issues

**Symptoms**: Connection timeouts, SSL errors

**Solutions**:
```kotlin
// Check SDK configuration
val configs = BuzzebeesSDK.instance().servicesConfigs()
Log.d("SDK", "Base URL: ${configs.baseUrl}")
Log.d("SDK", "Timeout: ${configs.timeoutInterval}")

// Verify network permissions in AndroidManifest.xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

#### 4. Custom API Issues

**Symptoms**: Custom APIs not working, configuration errors

**Solutions**:
```kotlin
// Test basic SDK functionality first
val okHttpClient = BuzzebeesSDK.instance().getConfiguredOkHttpClient()
val retrofit = BuzzebeesSDK.instance().getConfiguredRetrofit()

// Verify custom API configuration
val customApi = BuzzebeesSDK.instance().customApiBuilder()
    .baseUrl("https://api.example.com/") // Ensure URL ends with /
    .build(CustomApi::class.java)

// Test with direct HTTP call
val request = Request.Builder()
    .url("https://api.example.com/test")
    .build()
    
okHttpClient.newCall(request).execute().use { response ->
    Log.d("CustomAPI", "Response: ${response.code}")
}
```

#### 5. Badge Updates Not Working

**Symptoms**: Badge updates not received, listener not triggered

**Solutions**:
```kotlin
// Check badge manager registration
val badgeManager = BuzzebeesSDK.instance().badgeManager
val listener = object : BadgeListener {
    override fun onBadgeReceived(badgeModel: BadgeTraceModel) {
        Log.d("Badge", "Received badge: ${badgeModel.count}")
    }
}

// Ensure proper lifecycle management
lifecycleScope.launch {
    badgeManager.registerBadgeListener(listener)
}

// Don't forget to unregister
override fun onDestroy() {
    super.onDestroy()
    badgeManager.unregisterBadgeListener(listener)
}

// Check badge data directly
lifecycleScope.launch {
    badgeManager.getBadgeModelFlow().collect { badgeModel ->
        Log.d("Badge", "Badge model: $badgeModel")
    }
}
```

### Debug Logging

Enable comprehensive logging for troubleshooting:

```kotlin
class DebugHelper {
    fun enableSDKDebugging() {
        // Check SDK state
        val sdk = BuzzebeesSDK.instance()
        val configs = sdk.servicesConfigs()
        val authProvider = sdk.authProvider()
        val deviceInfo = sdk.deviceInfoProvider()
        
        Log.d("SDK_DEBUG", "Base URL: ${configs.baseUrl}")
        Log.d("SDK_DEBUG", "App ID: ${configs.appId}")
        Log.d("SDK_DEBUG", "Has Auth Token: ${authProvider.hasAuthToken()}")
        Log.d("SDK_DEBUG", "Device Info: ${deviceInfo.getDeviceInfo()}")
        
        // Test custom API configuration
        val okHttpClient = sdk.getConfiguredOkHttpClient()
        Log.d("SDK_DEBUG", "OkHttp interceptors: ${okHttpClient.interceptors.size}")
        
        val retrofit = sdk.getConfiguredRetrofit()
        Log.d("SDK_DEBUG", "Retrofit base URL: ${retrofit.baseUrl()}")
    }
}
```

---

## Conclusion

The BuzzebeesSDK provides a comprehensive platform for building feature-rich mobile applications with:

### Core Strengths

- **Complete API Coverage**: 22+ use cases covering all business domains
- **Custom API Integration**: Leverage SDK infrastructure for custom APIs
- **Real-time Badge Management**: Lifecycle-aware badge updates
- **Comprehensive Authentication**: Multi-token management with locale support
- **URL Building**: Automated URL generation for campaigns and profiles
- **Error Handling**: Production-ready error management framework
- **Performance Optimized**: Shared resources and connection pooling

### Development Benefits

- **Reduced Development Time**: Pre-built use cases and infrastructure
- **Consistent Configuration**: Centralized HTTP and authentication setup
- **Easy Integration**: Simple APIs with comprehensive documentation
- **Extensible Architecture**: Add custom functionality while leveraging SDK benefits
- **Production Ready**: Battle-tested error handling and lifecycle management

### Migration Path

For teams migrating from manual HTTP implementations:

1. **Start Small**: Begin with simple use cases like authentication
2. **Leverage Custom APIs**: Gradually migrate existing APIs to SDK infrastructure
3. **Adopt URL Extensions**: Replace manual URL construction
4. **Implement Badge Management**: Add real-time features
5. **Enhance Error Handling**: Adopt SDK error handling patterns

The SDK is designed to grow with your application, providing both simple entry points for basic functionality and advanced features for complex use cases. Whether you're building a new application or enhancing an existing one, the BuzzebeesSDK provides the foundation for scalable, maintainable, and feature-rich mobile experiences.

For specific implementation guidance, refer to the individual guides in the `/readme/` directory, and explore the comprehensive examples in `/examples/CustomApiExamples.kt`.
