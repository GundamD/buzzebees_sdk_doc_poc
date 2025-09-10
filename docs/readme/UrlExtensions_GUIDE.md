# URL Builder Extensions Guide

This guide provides comprehensive documentation for using the BuzzebeesSDK URL building capabilities through the `BuzzebeesSDK_UrlExtension.kt` file. These extensions allow you to build URLs for campaigns, profile images, redemption flows, shopping cart access, and generic URL building with automatic configuration and parameter handling.

---

## Table of Contents

1. [Overview](#overview)
2. [Available Extension Functions](#available-extension-functions)
3. [Generic URL Building](#generic-url-building)
4. [Access Token Management](#access-token-management)
5. [Shopping Cart Extensions](#shopping-cart-extensions)
6. [Usage Examples](#usage-examples)
7. [Integration Patterns](#integration-patterns)
8. [Best Practices](#best-practices)
9. [Troubleshooting](#troubleshooting)

---

## Overview

The URL Builder Extensions provide convenient methods for:

- **Profile Image URLs**: Generate complete URLs for user profile images
- **Campaign Image URLs**: Build full image URLs for campaigns with proper base URL handling
- **Generic URL Building**: 🆕 Flexible URL building with template replacement and parameter handling
- **Redeem URLs**: Generate complete redemption URLs with template replacements and parameters  
- **Access Token Management**: 🆕 Generic access token generation for any purpose
- **Shopping Cart URLs**: Build shopping cart URLs with access token management and parameter handling

### Benefits

✅ **Automatic Configuration**: Uses SDK's configured base URLs and authentication  
✅ **Template Processing**: 🆕 Flexible template replacement with configurable patterns  
✅ **Parameter Management**: Enhanced query parameter and URL option handling  
✅ **Type Safety**: Strongly typed parameters with proper validation  
✅ **Generic Access Tokens**: 🆕 Reusable access token system for multiple services  
✅ **Easy Integration**: Simple extension functions on the main SDK instance  

---

## Available Extension Functions

### 1. Profile Image URL Builder

**Function**: `buildProfileImageUrl(userId: String): String`

Builds a complete profile image URL for the specified user.

**Example**:
```kotlin
val profileImageUrl = BuzzebeesSDK.instance().buildProfileImageUrl("user123")
// Returns: "https://api.buzzebees.com/profiles/user123/picture?token=auto_token"
```

### 2. Full Image URL Builder

**Function**: `buildFullImageUrl(image: String? = null, id: String? = ""): String`

Builds complete image URLs for campaigns with proper base URL handling.

**Example**:
```kotlin
val imageUrl = BuzzebeesSDK.instance().buildFullImageUrl(
    image = "campaign-banner.jpg",
    id = "campaign_123"
)
```

### 3. Redeem URL Builder

**Function**: `buildRedeemUrl(campaign: Campaign?, userId: String, options: RedeemOptions = RedeemOptions(), urlOptions: UrlBuildOptions = UrlBuildOptions(), templateConfig: TemplateReplacementConfig = TemplateReplacementConfig()): String`

**Function**: `buildRedeemUrl(campaignDetails: CampaignDetails?, userId: String, options: RedeemOptions = RedeemOptions(), urlOptions: UrlBuildOptions = UrlBuildOptions(), templateConfig: TemplateReplacementConfig = TemplateReplacementConfig()): String`

Builds complete redemption URLs with enhanced template replacement and parameter handling.

**Example**:
```kotlin
val redeemOptions = RedeemOptions(
    color = "red",
    size = "L", 
    quantity = "2",
    walletCard = "visa123"
)

val urlOptions = UrlBuildOptions(
    shouldShuffleToken = true,
    nativeHeader = false,
    header = true,
    additionalParams = mapOf(
        "customParam1" to "value1",
        "isAllianzMember" to "N"
    )
)

val templateConfig = TemplateReplacementConfig(
    tokenPatterns = listOf("{token}", "<token>", "{auth_token}"),
    customReplacements = mapOf(
        "{special_param}" to "special_value"
    )
)

val redeemUrl = BuzzebeesSDK.instance().buildRedeemUrl(
    campaign = campaign,
    userId = "user123",
    options = redeemOptions,
    urlOptions = urlOptions,
    templateConfig = templateConfig
)
```

---

## Generic URL Building

### 🆕 Build Generic URL with Template Replacement

**Function**: `buildGenericUrl(baseUrl: String, urlOptions: UrlBuildOptions = UrlBuildOptions(), templateConfig: TemplateReplacementConfig = TemplateReplacementConfig(), userId: String? = null, campaignId: String? = null): String`

The most flexible URL builder that can enhance any URL with template replacement and parameter handling.

**Data Classes**:

```kotlin
data class UrlBuildOptions(
    val shouldShuffleToken: Boolean = true,
    val nativeHeader: Boolean = false,
    val header: Boolean = true,
    val additionalParams: Map<String, String> = emptyMap()
)

data class TemplateReplacementConfig(
    val tokenPatterns: List<String> = listOf("{token}", "<token>"),
    val userIdPatterns: List<String> = listOf("{userId}", "<userId>", "{user_id}", "<user_id>"),
    val campaignIdPatterns: List<String> = listOf("{campaignId}", "<campaignId>", "{CampaignId}", "{campaign_id}"),
    val customReplacements: Map<String, String> = emptyMap()
)
```

**Examples**:

```kotlin
// Basic URL enhancement
var url = "https://example.com/{token}/campaign?id={campaignId}"
val enhancedUrl = BuzzebeesSDK.instance().buildGenericUrl(
    baseUrl = url,
    userId = "user123",
    campaignId = "campaign456"
)
// Result: "https://example.com/shuffled_token_here/campaign?id=campaign456&token=shuffled_token_here&return_url=myapp&appName=MyApp&theme=MyApp&nativeheader=false&header=true"

// Advanced configuration
var url = bannerComponentItemModel.url.value()
val enhancedUrl = BuzzebeesSDK.instance().buildGenericUrl(
    baseUrl = url,
    urlOptions = UrlBuildOptions(
        shouldShuffleToken = false, // Use original token
        nativeHeader = false,
        header = true,
        additionalParams = mapOf(
            "appName" to appName,
            "theme" to appName,
            "isAllianzMember" to "N"
        )
    ),
    templateConfig = TemplateReplacementConfig(
        tokenPatterns = listOf("{token}", "<token>"),
        customReplacements = mapOf(
            "{locale}" to "th",
            "{special_param}" to "custom_value"
        )
    )
)

// Replace both {token} and <token> patterns
var url = "https://api.com/service?auth={token}&backup=<token>&user={userId}"
val processedUrl = BuzzebeesSDK.instance().buildGenericUrl(
    baseUrl = url,
    userId = "user123"
)
```

**Template Replacement Features**:
- **Token Patterns**: `{token}`, `<token>` - Replaced with authentication token
- **User ID Patterns**: `{userId}`, `<userId>`, `{user_id}`, `<user_id>`
- **Campaign ID Patterns**: `{campaignId}`, `<campaignId>`, `{CampaignId}`, `{campaign_id}`
- **Device Info**: `<uid>`, `<imei>`, `<deviceId>`, `<model>`, `{model}`, `<device_locale>`, `{device_locale}`
- **Custom Replacements**: Any custom pattern you define

**URL Options**:
- **shouldShuffleToken**: Control token shuffling behavior
- **nativeHeader**: Native header visibility
- **header**: Standard header visibility  
- **additionalParams**: Custom URL parameters

---

## Access Token Management

### 🆕 Generic Access Token System

The SDK now provides a generic access token system that can be used for shopping carts and other services.

#### Access Token Results

```kotlin
sealed class AccessTokenResult {
    data class Success(val accessToken: String) : AccessTokenResult()
    data class Error(val message: String, val throwable: Throwable? = null) : AccessTokenResult()
}
```

#### Request Access Token

**Function**: `requestAccessToken(request: AccessTokenRequest): AccessTokenResult`

Generic method to get access tokens for any purpose.

```kotlin
suspend fun getAccessToken() {
    val request = BuzzebeesSDK.instance().generateAccessTokenRequest()
    
    when (val result = BuzzebeesSDK.instance().requestAccessToken(request)) {
        is AccessTokenResult.Success -> {
            val accessToken = result.accessToken
            // Use token for any service
            useTokenForCustomService(accessToken)
        }
        is AccessTokenResult.Error -> {
            // Handle error
            showError(result.message)
        }
    }
}
```

#### Generate Access Token Requests

**Function**: `generateAccessTokenRequest(customReturnUrl: String? = null, appId: String? = null, appName: String? = null): AccessTokenRequest`

Generate request using SDK configuration.

**Function**: `generateCustomAccessTokenRequest(errorUrl: String, successUrl: String, returnUrl: String, appId: String, appName: String): AccessTokenRequest`

Generate request with custom URLs.

```kotlin
// Using SDK configuration
val request = BuzzebeesSDK.instance().generateAccessTokenRequest()

// Custom configuration  
val customRequest = BuzzebeesSDK.instance().generateCustomAccessTokenRequest(
    errorUrl = "myapp://error",
    successUrl = "myapp://success", 
    returnUrl = "myapp://return",
    appId = "my-app-id",
    appName = "MyApp"
)
```

---

## Shopping Cart Extensions

### Shopping Cart URL Building

The shopping cart system now uses the generic access token system and supports enhanced URL options.

#### Data Classes

```kotlin
data class ShoppingCartUrlResult(
    val url: String,
    val accessToken: String
)

sealed class ShoppingCartUrlBuildResult {
    data class Success(val result: ShoppingCartUrlResult) : ShoppingCartUrlBuildResult()
    data class Error(val message: String, val throwable: Throwable? = null) : ShoppingCartUrlBuildResult()
}
```

#### Main Functions

**Function**: `buildShoppingCartUrl(request: AccessTokenRequest, baseCartUrl: String? = null, urlOptions: UrlBuildOptions = UrlBuildOptions()): ShoppingCartUrlBuildResult`

Build shopping cart URL with automatic access token retrieval.

**Function**: `buildDefaultShoppingCartUrl(customReturnUrl: String? = null, baseCartUrl: String? = null): ShoppingCartUrlBuildResult`

Build cart URL using SDK defaults.

**Function**: `buildFlexibleCartUrl(accessToken: String, path: String? = null, additionalParams: Map<String, String> = emptyMap(), appName: String? = null, baseCartUrl: String? = null): String`

Build specific cart URLs with existing token.

#### Examples

```kotlin
class ShoppingCartManager {
    
    // Simple cart access
    suspend fun openCart(): String? {
        val result = BuzzebeesSDK.instance().buildDefaultShoppingCartUrl()
        
        return when (result) {
            is ShoppingCartUrlBuildResult.Success -> {
                // Save token for future use
                saveAccessToken(result.result.accessToken)
                result.result.url
            }
            is ShoppingCartUrlBuildResult.Error -> {
                Log.e("Cart", result.message)
                null
            }
        }
    }
    
    // Advanced cart access with options
    suspend fun openCartAdvanced(): String? {
        val request = BuzzebeesSDK.instance().generateAccessTokenRequest(
            customReturnUrl = "myapp://cart-return"
        )
        
        val urlOptions = UrlBuildOptions(
            additionalParams = mapOf(
                "theme" to "dark",
                "language" to "th"
            )
        )
        
        val result = BuzzebeesSDK.instance().buildShoppingCartUrl(request, urlOptions = urlOptions)
        
        return when (result) {
            is ShoppingCartUrlBuildResult.Success -> result.result.url
            is ShoppingCartUrlBuildResult.Error -> null
        }
    }
    
    // Navigate to specific cart pages
    fun navigateToMyOrders(accessToken: String): String {
        return BuzzebeesSDK.instance().buildFlexibleCartUrl(
            accessToken = accessToken,
            path = "/myorder",
            additionalParams = mapOf("sortBy" to "date")
        )
    }
    
    fun navigateToVouchers(accessToken: String, category: String? = null): String {
        val params = category?.let { mapOf("category" to it) } ?: emptyMap()
        return BuzzebeesSDK.instance().buildFlexibleCartUrl(
            accessToken = accessToken,
            path = "/voucher",
            additionalParams = params
        )
    }
}
```

#### Helper Functions

**Function**: `isShoppingCartAvailable(): Boolean`

Check if shopping cart configuration is available.

```kotlin
if (BuzzebeesSDK.instance().isShoppingCartAvailable()) {
    // All required config is present
    openCart()
} else {
    // Show configuration error
}
```

---

## Usage Examples

### Generic URL Building in Activities

```kotlin
class CampaignDetailActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        val bannerUrl = intent.getStringExtra("banner_url") ?: return
        val userId = getCurrentUserId()
        val campaignId = intent.getStringExtra("campaign_id")
        
        // Enhance any URL with template replacement
        val enhancedUrl = BuzzebeesSDK.instance().buildGenericUrl(
            baseUrl = bannerUrl,
            urlOptions = UrlBuildOptions(
                shouldShuffleToken = true,
                additionalParams = mapOf(
                    "appName" to BuildConfig.APP_NAME,
                    "version" to BuildConfig.VERSION_NAME,
                    "platform" to "android"
                )
            ),
            templateConfig = TemplateReplacementConfig(
                customReplacements = mapOf(
                    "{locale}" to Locale.getDefault().language,
                    "{theme}" to getAppTheme()
                )
            ),
            userId = userId,
            campaignId = campaignId
        )
        
        openWebView(enhancedUrl)
    }
}
```

### Advanced Redeem URL Building

```kotlin
class RedeemActivity : AppCompatActivity() {
    
    private fun buildAdvancedRedeemUrl(campaign: Campaign): String {
        val redeemOptions = RedeemOptions(
            color = colorSpinner.selectedItem?.toString(),
            size = sizeSpinner.selectedItem?.toString(),
            quantity = quantityPicker.value.toString(),
            walletCard = getSelectedWalletCard()
        )
        
        val urlOptions = UrlBuildOptions(
            shouldShuffleToken = preferences.getBoolean("use_shuffled_token", true),
            nativeHeader = false,
            header = !preferences.getBoolean("hide_header", false),
            additionalParams = mapOf(
                "source" to "mobile_app",
                "version" to BuildConfig.VERSION_NAME,
                "isAllianzMember" to if (isAllianzMember()) "Y" else "N",
                "deviceType" to "smartphone"
            )
        )
        
        val templateConfig = TemplateReplacementConfig(
            tokenPatterns = listOf("{token}", "<token>", "{auth_token}"),
            customReplacements = mapOf(
                "{promo_code}" to getCurrentPromoCode(),
                "{referral_id}" to getReferralId(),
                "{session_id}" to getSessionId()
            )
        )
        
        return BuzzebeesSDK.instance().buildRedeemUrl(
            campaign = campaign,
            userId = getCurrentUserId(),
            options = redeemOptions,
            urlOptions = urlOptions,
            templateConfig = templateConfig
        )
    }
}
```

### Shopping Cart with Token Management

```kotlin
class ShoppingViewModel : ViewModel() {
    private var accessToken: String? = null
    
    suspend fun initializeShoppingCart(): Boolean {
        return try {
            val result = BuzzebeesSDK.instance().buildDefaultShoppingCartUrl()
            
            when (result) {
                is ShoppingCartUrlBuildResult.Success -> {
                    accessToken = result.result.accessToken
                    _cartUrl.value = result.result.url
                    true
                }
                is ShoppingCartUrlBuildResult.Error -> {
                    _error.value = result.message
                    false
                }
            }
        } catch (e: Exception) {
            _error.value = "Failed to initialize shopping cart"
            false
        }
    }
    
    fun navigateToSection(section: String) {
        val token = accessToken ?: return
        
        val url = when (section) {
            "myorders" -> BuzzebeesSDK.instance().buildFlexibleCartUrl(
                accessToken = token,
                path = "/myorder",
                additionalParams = mapOf("tab" to "active")
            )
            "vouchers" -> BuzzebeesSDK.instance().buildFlexibleCartUrl(
                accessToken = token,
                path = "/voucher"
            )
            "settings" -> BuzzebeesSDK.instance().buildFlexibleCartUrl(
                accessToken = token,
                path = "/settings"
            )
            else -> BuzzebeesSDK.instance().buildFlexibleCartUrl(token)
        }
        
        _navigationUrl.value = url
    }
}
```

---

## Integration Patterns

### Pattern 1: URL Builder Service

```kotlin
class UrlBuilderService {
    private val sdk = BuzzebeesSDK.instance()
    
    // Generic URL enhancement
    fun enhanceUrl(
        baseUrl: String,
        userId: String? = null,
        campaignId: String? = null,
        additionalParams: Map<String, String> = emptyMap(),
        customReplacements: Map<String, String> = emptyMap()
    ): String {
        return sdk.buildGenericUrl(
            baseUrl = baseUrl,
            urlOptions = UrlBuildOptions(
                additionalParams = additionalParams
            ),
            templateConfig = TemplateReplacementConfig(
                customReplacements = customReplacements
            ),
            userId = userId,
            campaignId = campaignId
        )
    }
    
    // Specialized redeem URL
    fun buildRedeemUrl(
        campaign: Campaign,
        userId: String,
        options: Map<String, String> = emptyMap()
    ): String {
        val redeemOptions = RedeemOptions(
            color = options["color"],
            size = options["size"],
            quantity = options["quantity"],
            walletCard = options["wallet_card"]
        )
        
        return sdk.buildRedeemUrl(
            campaign = campaign,
            userId = userId,
            options = redeemOptions
        )
    }
    
    // Access token management
    suspend fun getAccessToken(): String? {
        val request = sdk.generateAccessTokenRequest()
        return when (val result = sdk.requestAccessToken(request)) {
            is AccessTokenResult.Success -> result.accessToken
            is AccessTokenResult.Error -> null
        }
    }
}
```

### Pattern 2: Repository Pattern with Enhanced URLs

```kotlin
interface CampaignRepository {
    suspend fun getCampaignUrls(campaignId: String, userId: String): CampaignUrls
    suspend fun buildCustomUrl(baseUrl: String, options: UrlOptions): String
}

data class CampaignUrls(
    val imageUrl: String,
    val redeemUrl: String,
    val websiteUrl: String
)

data class UrlOptions(
    val additionalParams: Map<String, String> = emptyMap(),
    val customReplacements: Map<String, String> = emptyMap(),
    val shouldShuffleToken: Boolean = true
)

class CampaignRepositoryImpl : CampaignRepository {
    private val sdk = BuzzebeesSDK.instance()
    
    override suspend fun getCampaignUrls(campaignId: String, userId: String): CampaignUrls {
        val campaign = getCampaign(campaignId)
        
        return CampaignUrls(
            imageUrl = sdk.buildFullImageUrl(campaign.imageUrl, campaign.id),
            redeemUrl = sdk.buildRedeemUrl(campaign, userId),
            websiteUrl = campaign.websiteUrl // Or use buildGenericUrl for enhancement
        )
    }
    
    override suspend fun buildCustomUrl(baseUrl: String, options: UrlOptions): String {
        return sdk.buildGenericUrl(
            baseUrl = baseUrl,
            urlOptions = UrlBuildOptions(
                shouldShuffleToken = options.shouldShuffleToken,
                additionalParams = options.additionalParams
            ),
            templateConfig = TemplateReplacementConfig(
                customReplacements = options.customReplacements
            )
        )
    }
}
```

---

## Best Practices

### 1. Use Generic URL Builder for Flexibility

```kotlin
// Instead of manual URL construction
var url = baseUrl
url = url.replace("{token}", getToken())
url = url.replace("{userId}", userId)
url += "&appName=" + getAppName()
url += "&theme=" + getTheme()

// Use generic URL builder
val enhancedUrl = BuzzebeesSDK.instance().buildGenericUrl(
    baseUrl = baseUrl,
    urlOptions = UrlBuildOptions(
        additionalParams = mapOf(
            "appName" to getAppName(),
            "theme" to getTheme()
        )
    ),
    userId = userId
)
```

### 2. Cache Access Tokens

```kotlin
class TokenManager {
    private var cachedToken: String? = null
    private var tokenExpiry: Long = 0
    
    suspend fun getValidToken(): String? {
        if (isTokenValid()) {
            return cachedToken
        }
        
        return when (val result = BuzzebeesSDK.instance().requestAccessToken(generateRequest())) {
            is AccessTokenResult.Success -> {
                cachedToken = result.accessToken
                tokenExpiry = System.currentTimeMillis() + TimeUnit.HOURS.toMillis(2)
                result.accessToken
            }
            is AccessTokenResult.Error -> null
        }
    }
    
    private fun isTokenValid(): Boolean {
        return cachedToken != null && System.currentTimeMillis() < tokenExpiry
    }
}
```

### 3. Configure Template Replacements Globally

```kotlin
object UrlConfig {
    fun getStandardTemplateConfig(): TemplateReplacementConfig {
        return TemplateReplacementConfig(
            tokenPatterns = listOf("{token}", "<token>", "{auth_token}"),
            customReplacements = mapOf(
                "{app_version}" to BuildConfig.VERSION_NAME,
                "{platform}" to "android",
                "{locale}" to Locale.getDefault().language
            )
        )
    }
    
    fun getStandardUrlOptions(): UrlBuildOptions {
        return UrlBuildOptions(
            shouldShuffleToken = true,
            nativeHeader = false,
            header = true,
            additionalParams = mapOf(
                "source" to "mobile_app",
                "platform" to "android"
            )
        )
    }
}

// Usage
val enhancedUrl = BuzzebeesSDK.instance().buildGenericUrl(
    baseUrl = baseUrl,
    urlOptions = UrlConfig.getStandardUrlOptions(),
    templateConfig = UrlConfig.getStandardTemplateConfig(),
    userId = userId
)
```

### 4. Error Handling

```kotlin
suspend fun buildUrlSafely(baseUrl: String, userId: String?): String? {
    return try {
        BuzzebeesSDK.instance().buildGenericUrl(
            baseUrl = baseUrl,
            userId = userId
        )
    } catch (e: Exception) {
        Log.e("UrlBuilder", "Failed to build URL", e)
        null
    }
}

suspend fun getAccessTokenSafely(): String? {
    return try {
        val request = BuzzebeesSDK.instance().generateAccessTokenRequest()
        when (val result = BuzzebeesSDK.instance().requestAccessToken(request)) {
            is AccessTokenResult.Success -> result.accessToken
            is AccessTokenResult.Error -> {
                Log.e("Token", "Failed to get access token: ${result.message}")
                null
            }
        }
    } catch (e: Exception) {
        Log.e("Token", "Exception getting access token", e)
        null
    }
}
```

---

## Troubleshooting

### Common Issues and Solutions

#### 1. Template Replacement Not Working

**Problem**: Custom patterns like `{custom_param}` not being replaced

**Solution**:
```kotlin
val templateConfig = TemplateReplacementConfig(
    customReplacements = mapOf(
        "{custom_param}" to "custom_value",
        "<custom_param>" to "custom_value" // Support both patterns
    )
)

val url = BuzzebeesSDK.instance().buildGenericUrl(
    baseUrl = baseUrl,
    templateConfig = templateConfig
)
```

#### 2. Access Token Request Fails

**Problem**: `requestAccessToken` returns error

**Solution**:
```kotlin
// Check SDK configuration
if (!BuzzebeesSDK.instance().isShoppingCartAvailable()) {
    // Missing configuration
    Log.e("Token", "Shopping cart configuration missing")
}

// Verify network connectivity
// Check appId and appName in SDK configuration
val configs = BuzzebeesSDK.instance().servicesConfigs()
Log.d("Token", "App ID: ${configs.appId}, App Name: ${configs.appName}")
```

#### 3. Token Shuffling Issues

**Problem**: Token shuffling not working as expected

**Solution**:
```kotlin
// Disable token shuffling if needed
val urlOptions = UrlBuildOptions(
    shouldShuffleToken = false // Use original token
)

// Or debug token shuffling
val originalToken = BuzzebeesSDK.instance().authProvider().getAuthToken()
Log.d("Token", "Original token: $originalToken")

val enhancedUrl = BuzzebeesSDK.instance().buildGenericUrl(
    baseUrl = baseUrl,
    urlOptions = urlOptions
)
```

#### 4. Shopping Cart URLs Not Working

**Problem**: Cart URLs return 404 or access denied

**Solution**:
```kotlin
// Verify base cart URL configuration
val configs = BuzzebeesSDK.instance().servicesConfigs()
Log.d("Cart", "Web shopping URL: ${configs.webShoppingUrl}")

// Check access token validity
suspend fun validateToken(token: String): Boolean {
    // Try to use token in a simple cart API call
    // Return true if successful, false otherwise
}

// Use fresh token
val result = BuzzebeesSDK.instance().buildDefaultShoppingCartUrl()
when (result) {
    is ShoppingCartUrlBuildResult.Success -> {
        if (validateToken(result.result.accessToken)) {
            // Token is valid
        }
    }
    is ShoppingCartUrlBuildResult.Error -> {
        // Handle error
    }
}
```

---

## Summary of Changes

### 🆕 New Features

1. **Generic URL Builder**: `buildGenericUrl()` for flexible URL enhancement
2. **Template Replacement Config**: Configurable patterns and custom replacements
3. **URL Build Options**: Enhanced parameter handling and token control
4. **Generic Access Token System**: Reusable access token management
5. **Enhanced Shopping Cart**: Better URL options and error handling

### ✅ Improved Features

1. **Flexible Template Patterns**: Support for both `{pattern}` and `<pattern>` styles
2. **Enhanced Parameter Management**: More control over URL parameters
3. **Better Error Handling**: Comprehensive error types and messages
4. **Token Control**: Choose between shuffled and original tokens
5. **Backward Compatibility**: All existing functionality preserved

### 🔧 Migration Guide

**Old Code**:
```kotlin
// Manual URL building
var url = baseUrl.replace("{token}", token)
url += "&appName=" + appName
```

**New Code**:
```kotlin
// Generic URL builder
val url = BuzzebeesSDK.instance().buildGenericUrl(
    baseUrl = baseUrl,
    urlOptions = UrlBuildOptions(
        additionalParams = mapOf("appName" to appName)
    )
)
```

The new URL building system provides maximum flexibility while maintaining backward compatibility with existing code.

---

## Conclusion

The enhanced URL Builder Extensions provide powerful and flexible URL generation for all Buzzebees services. The new generic URL builder and access token management system offer unprecedented flexibility while maintaining ease of use.

Key advantages:
- **🆕 Generic URL Building**: Handle any URL with template replacement
- **🆕 Flexible Access Tokens**: Reusable token system for multiple services
- **Enhanced Template Processing**: Configurable patterns and replacements
- **Better Parameter Management**: More control over URL parameters
- **Improved Error Handling**: Comprehensive error reporting
- **Backward Compatibility**: All existing functionality preserved

For additional implementation details, refer to the `BuzzebeesSDK_UrlExtension.kt` source file.
