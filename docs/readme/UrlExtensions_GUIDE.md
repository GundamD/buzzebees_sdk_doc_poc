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
7. [Best Practices](#best-practices)

---

## Overview

The URL Builder Extensions provide convenient methods for:

- **Profile Image URLs**: Generate complete URLs for user profile images
- **Campaign Image URLs**: Build full image URLs for campaigns with proper base URL handling
- **Generic URL Building**: Flexible URL building with template replacement and parameter handling
- **Redeem URLs**: Generate complete redemption URLs with template replacements and parameters  
- **Access Token Management**: Generic access token generation for any purpose
- **Shopping Cart URLs**: Build shopping cart URLs with access token management and parameter handling

### Security Architecture

The URL building system uses a secure architecture pattern:
- **Interface-Based**: All functionality exposed through `IUrlHandler` interface
- **Implementation Hidden**: Business logic secured in `UrlHandlerImpl` (obfuscated)
- **Extension Pattern**: Clean SDK integration via extension functions
- **Configuration Secured**: All sensitive configs handled internally

### Benefits

✅ **Automatic Configuration**: Uses SDK's configured base URLs and authentication  
✅ **Template Processing**: Flexible template replacement with configurable patterns  
✅ **Parameter Management**: Enhanced query parameter and URL option handling  
✅ **Type Safety**: Strongly typed parameters with proper validation  
✅ **Generic Access Tokens**: Reusable access token system for multiple services  
✅ **Easy Integration**: Simple extension functions on the main SDK instance  
✅ **Secure by Design**: Business logic hidden from clients through interface pattern

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

### Build Generic URL with Template Replacement

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
val url = "https://example.com/{token}/campaign?id={campaignId}"
val enhancedUrl = BuzzebeesSDK.instance().buildGenericUrl(
    baseUrl = url,
    userId = "user123",
    campaignId = "campaign456"
)

// Advanced configuration
val enhancedUrl = BuzzebeesSDK.instance().buildGenericUrl(
    baseUrl = bannerUrl,
    urlOptions = UrlBuildOptions(
        shouldShuffleToken = false,
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
```

---

## Access Token Management

### Generic Access Token System

The SDK provides a generic access token system that can be used for shopping carts and other services.

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

#### Validation

**Function**: `validateAccessTokenRequest(request: AccessTokenRequest): Boolean`

Validate access token request before use.

```kotlin
val request = BuzzebeesSDK.instance().generateAccessTokenRequest()
if (BuzzebeesSDK.instance().validateAccessTokenRequest(request)) {
    // Request is valid, proceed
    val result = BuzzebeesSDK.instance().requestAccessToken(request)
}
```

---

## Shopping Cart Extensions

### Shopping Cart URL Building

The shopping cart system uses the generic access token system and supports enhanced URL options.

#### Data Classes

```kotlin
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
                    "version" to BuildConfig.VERSION_NAME
                )
            ),
            templateConfig = TemplateReplacementConfig(
                customReplacements = mapOf(
                    "{locale}" to Locale.getDefault().language
                )
            ),
            userId = userId,
            campaignId = campaignId
        )
        
        openWebView(enhancedUrl)
    }
}
```

### Shopping Cart with Token Management

```kotlin
class ShoppingViewModel : ViewModel() {
    
    suspend fun initializeShoppingCart(): Boolean {
        return try {
            val result = BuzzebeesSDK.instance().buildDefaultShoppingCartUrl()
            
            when (result) {
                is ShoppingCartUrlBuildResult.Success -> {
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
    
    fun navigateToSection(accessToken: String, section: String) {
        val url = when (section) {
            "myorders" -> BuzzebeesSDK.instance().buildFlexibleCartUrl(
                accessToken = accessToken,
                path = "/myorder"
            )
            "vouchers" -> BuzzebeesSDK.instance().buildFlexibleCartUrl(
                accessToken = accessToken,
                path = "/voucher"
            )
            else -> BuzzebeesSDK.instance().buildFlexibleCartUrl(accessToken)
        }
        
        _navigationUrl.value = url
    }
}
```

---

## Best Practices

### 1. Use Generic URL Builder for Flexibility

```kotlin
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

### 2. Validate Requests Before Use

```kotlin
val request = BuzzebeesSDK.instance().generateAccessTokenRequest()
if (BuzzebeesSDK.instance().validateAccessTokenRequest(request)) {
    val result = BuzzebeesSDK.instance().requestAccessToken(request)
    // Use result
}
```

### 3. Check Availability Before Use

```kotlin
if (BuzzebeesSDK.instance().isShoppingCartAvailable()) {
    // Proceed with cart functionality
} else {
    // Handle missing configuration
}
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
```

---

## Summary

The URL Builder Extensions provide secure and flexible URL generation for all Buzzebees services through a clean interface-based architecture. The system offers:

**Core Features:**
- **Profile & Campaign URLs**: Complete image URL generation
- **Generic URL Building**: Flexible template replacement system  
- **Access Token Management**: Reusable token system for multiple services
- **Shopping Cart Integration**: Complete cart URL building with token management
- **Template Processing**: Configurable patterns and custom replacements

**Security Benefits:**
- **Interface-Based Design**: Business logic hidden through `IUrlHandler` interface
- **Implementation Security**: All sensitive logic in obfuscated `UrlHandlerImpl`
- **Configuration Protection**: SDK configs handled internally
- **Type Safety**: Strongly typed parameters with validation

**Key Advantages:**
- **Easy Integration**: Simple extension functions on SDK instance
- **Automatic Configuration**: Uses SDK's configured URLs and authentication
- **Enhanced Error Handling**: Comprehensive error types and validation
- **Backward Compatibility**: All existing functionality preserved
- **Flexible Parameters**: Enhanced control over URL generation

For implementation details, all business logic is secured within the `UrlHandlerImpl` class and accessed through the `IUrlHandler` interface for maximum security and obfuscation.
