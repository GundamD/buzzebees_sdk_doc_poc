# URL Builder Extensions Guide

This guide provides comprehensive documentation for using the BuzzebeesSDK URL building capabilities through the `BuzzebeesSDK_UrlExtension.kt` file. These extensions allow you to build URLs for campaigns, profile images, redemption flows, and shopping cart access with automatic configuration and parameter handling.

---

## Table of Contents

1. [Overview](#overview)
2. [Accessing URL Builder](#accessing-url-builder)
3. [Available Extension Functions](#available-extension-functions)
4. [Usage Examples](#usage-examples)
5. [Integration Patterns](#integration-patterns)
6. [Best Practices](#best-practices)
7. [Troubleshooting](#troubleshooting)

---

## Overview

The URL Builder Extensions provide convenient methods for:

- **Profile Image URLs**: Generate complete URLs for user profile images
- **Campaign Image URLs**: Build full image URLs for campaigns with proper base URL handling
- **Website URLs**: Create simple website URLs with basic campaign configuration
- **Redeem URLs**: Generate complete redemption URLs with template replacements and parameters
- **Shopping Cart URLs**: Build shopping cart URLs with access token management and parameter handling

### Benefits

✅ **Automatic Configuration**: Uses SDK's configured base URLs and authentication  
✅ **Template Processing**: Handles URL template replacements automatically  
✅ **Parameter Management**: Manages query parameters and user-specific data  
✅ **Type Safety**: Strongly typed parameters with proper validation  
✅ **Easy Integration**: Simple extension functions on the main SDK instance  

---

## Accessing URL Builder

### Direct Access to URL Builder

```kotlin
val urlBuilder = BuzzebeesSDK.instance().urlBuilder
```

The URL builder automatically uses:
- **Configs**: From `servicesConfigs()` - includes base URLs and app configuration
- **Auth Provider**: From `authProvider()` - provides authentication tokens
- **Device Info**: From `deviceInfoProvider()` - includes device-specific information

### Extension Functions on BuzzebeesSDK

The extension functions provide URL building functionality:

```kotlin
val profileUrl = BuzzebeesSDK.instance().buildProfileImageUrl(userId)
val campaignImageUrl = BuzzebeesSDK.instance().buildFullImageUrl(image, campaignId)

// Website URLs
val websiteUrl = BuzzebeesSDK.instance().buildWebsiteUrl(campaign)
val websiteUrlFromDetails = BuzzebeesSDK.instance().buildWebsiteUrl(campaignDetails) // 🆕 CampaignDetails support

// Redeem URLs
val redeemUrl = BuzzebeesSDK.instance().buildRedeemUrl(campaign, userId, options)
val redeemUrlFromDetails = BuzzebeesSDK.instance().buildRedeemUrl(campaignDetails, userId, options) // 🆕 CampaignDetails support

// Shopping cart URLs (async)
val shoppingCartResult = BuzzebeesSDK.instance().buildShoppingCartUrl(request)
val defaultCartResult = BuzzebeesSDK.instance().buildDefaultShoppingCartUrl()

// Cart URLs using flexible builder with different paths
val cartUrl = BuzzebeesSDK.instance().buildFlexibleCartUrl(accessToken) // Root page
val myOrderUrl = BuzzebeesSDK.instance().buildFlexibleCartUrl(accessToken, "/myorder")
val voucherUrl = BuzzebeesSDK.instance().buildFlexibleCartUrl(accessToken, "/voucher")
val settingsUrl = BuzzebeesSDK.instance().buildFlexibleCartUrl(accessToken, "/settings")
```

---

## Available Extension Functions

### 1. Profile Image URL Builder

**Function**: `buildProfileImageUrl(userId: String): String`

Builds a complete profile image URL for the specified user.

**Parameters**:
- `userId`: User ID for the profile

**Returns**: Complete profile image URL with authentication token automatically included

**Example**:
```kotlin
val userId = "12345"
val profileImageUrl = BuzzebeesSDK.instance().buildProfileImageUrl(userId)
// Returns: "https://api.buzzebees.com/profiles/12345/image?token=auto_token"
```

### 2. Full Image URL Builder

**Function**: `buildFullImageUrl(image: String? = null, id: String? = ""): String`

Builds complete image URLs for campaigns with proper base URL handling.

**Parameters**:
- `image`: Optional existing image URL
- `id`: Optional campaign ID

**Returns**: Complete image URL with base URL automatically retrieved from configuration

**Example**:
```kotlin
val imageUrl = BuzzebeesSDK.instance().buildFullImageUrl(
    image = "campaign-banner.jpg",
    id = "campaign_123"
)
// Returns: "https://cdn.buzzebees.com/images/campaign-banner.jpg"
```

### 3. Website URL Builder

**Function**: `buildWebsiteUrl(campaign: Campaign?): String`

Builds simple website URLs with basic campaign configuration.

**Parameters**:
- `campaign`: Campaign object containing website configuration

**Returns**: Complete website URL

**Example**:
```kotlin
val campaign = Campaign(
    id = "123",
    websiteUrl = "https://example.com/campaign",
    // ... other campaign properties
)

val websiteUrl = BuzzebeesSDK.instance().buildWebsiteUrl(campaign)
// Returns: "https://example.com/campaign" (processed with any necessary parameters)
```

### 4. Redeem URL Builder

**Function**: `buildRedeemUrl(campaign: Campaign?, userId: String, options: RedeemOptions = RedeemOptions()): String`

Builds complete redemption URLs with all template replacements and parameters.

**Parameters**:
- `campaign`: Campaign object containing website and redemption details
- `userId`: User ID for the redemption
- `options`: Redeem options including color, size, quantity, etc.

**Returns**: Complete redemption URL with all parameters and template replacements

**Example**:
```kotlin
val redeemOptions = RedeemOptions(
    color = "red",
    size = "L",
    quantity = 2,
    customData = mapOf("promo" to "SUMMER2024")
)

val redeemUrl = BuzzebeesSDK.instance().buildRedeemUrl(
    campaign = campaign,
    userId = "user_123",
    options = redeemOptions
)
// Returns: Complete URL with all parameters processed
```

### 5. Shopping Cart URL Builders

The shopping cart URL builders provide comprehensive functionality for managing shopping cart access tokens and building various cart-related URLs.

#### 5.1. Main Shopping Cart URL Builder

**Function**: `buildShoppingCartUrl(request: AccessTokenRequest, baseCartUrl: String? = null): ShoppingCartUrlBuildResult`

Builds complete shopping cart URLs with automatic access token retrieval and URL construction.

**Parameters**:
- `request`: AccessTokenRequest containing app configuration and return URLs
- `baseCartUrl`: Optional custom base cart URL (uses `webShoppingUrl` from config if null)

**Returns**: `ShoppingCartUrlBuildResult` containing either success with URL and token, or error details

**Example**:
```kotlin
val request = AccessTokenRequest(
    errorUrl = "myapp://",
    successUrl = "myapp://",
    returnUrl = "myapp://",
    appId = "your-app-id",
    appName = "beesbenefit"
)

// Async operation
val result = BuzzebeesSDK.instance().buildShoppingCartUrl(request)
when (result) {
    is ShoppingCartUrlBuildResult.Success -> {
        val url = result.result.url
        val token = result.result.accessToken
        // Use URL and save token
    }
    is ShoppingCartUrlBuildResult.Error -> {
        // Handle error
    }
}
```

#### 5.2. Default Shopping Cart URL Builder

**Function**: `buildDefaultShoppingCartUrl(customReturnUrl: String? = null, baseCartUrl: String? = null): ShoppingCartUrlBuildResult`

Builds shopping cart URLs using SDK configuration automatically.

**Parameters**:
- `customReturnUrl`: Optional custom return URL (uses `urlSchemesMainProject` from config if null)
- `baseCartUrl`: Optional custom base cart URL (uses `webShoppingUrl` from config if null)

**Returns**: `ShoppingCartUrlBuildResult` with automatically configured request

**Example**:
```kotlin
// Uses all SDK configuration automatically
val result = BuzzebeesSDK.instance().buildDefaultShoppingCartUrl()

// Or with custom return URL
val result = BuzzebeesSDK.instance().buildDefaultShoppingCartUrl(
    customReturnUrl = "customapp://return"
)
```

#### 5.3. Flexible Cart URL Builder

**Function**: `buildFlexibleCartUrl(accessToken: String, path: String? = null, additionalParams: Map<String, String> = emptyMap(), appName: String? = null, baseCartUrl: String? = null): String`

Builds cart URLs with custom paths and parameters. This is the main function for building all cart-related URLs.

**Parameters**:
- `accessToken`: Access token for authentication
- `path`: Optional additional path (e.g., "/myorder", "/voucher", "/settings", "/profile")
- `additionalParams`: Optional additional parameters
- `appName`: Optional custom app name (uses SDK config if null)
- `baseCartUrl`: Optional custom base URL (uses SDK config if null)

**Examples**:
```kotlin
// Cart root page (no path)
val cartUrl = BuzzebeesSDK.instance().buildFlexibleCartUrl(accessToken)
// Result: https://api.com/landing/beesbenefit?access_key=token&appname_config=beesbenefit

// My Orders page
val myOrderUrl = BuzzebeesSDK.instance().buildFlexibleCartUrl(
    accessToken = accessToken,
    path = "/myorder"
)
// Result: https://api.com/landing/beesbenefit/myorder?access_key=token&appname_config=beesbenefit

// Vouchers page
val voucherUrl = BuzzebeesSDK.instance().buildFlexibleCartUrl(
    accessToken = accessToken,
    path = "/voucher"
)
// Result: https://api.com/landing/beesbenefit/voucher?access_key=token&appname_config=beesbenefit

// Custom settings page with additional parameters
val settingsUrl = BuzzebeesSDK.instance().buildFlexibleCartUrl(
    accessToken = accessToken,
    path = "/settings",
    additionalParams = mapOf("tab" to "profile", "theme" to "dark")
)
// Result: https://api.com/landing/beesbenefit/settings?access_key=token&appname_config=beesbenefit&tab=profile&theme=dark

// Different app with custom path
val customUrl = BuzzebeesSDK.instance().buildFlexibleCartUrl(
    accessToken = accessToken,
    appName = "customapp",
    path = "/dashboard"
)
// Result: https://api.com/landing/customapp/dashboard?access_key=token&appname_config=customapp
```

**Common Paths You Can Use**:
- `null` or empty - Root cart page
- `"/myorder"` - My orders page
- `"/voucher"` - Vouchers/coupons page
- `"/settings"` - Settings page
- `"/profile"` - Profile page
- `"/history"` - Purchase history
- `"/support"` - Support/help page
- Any custom path your cart application supports

#### 5.4. Helper Functions

**Create AccessTokenRequest**: `createAccessTokenRequest(customReturnUrl: String? = null, appId: String? = null, appName: String? = null): AccessTokenRequest`

Creates AccessTokenRequest using SDK configuration.

**Check Availability**: `isShoppingCartAvailable(): Boolean`

Checks if all required configuration is available for shopping cart functionality.

**Validate AccessTokenRequest**: `validateAccessTokenRequest(request: AccessTokenRequest): Boolean`

Validates if an AccessTokenRequest has all required fields.

**Examples**:
```kotlin
// Create request with SDK defaults
val request = BuzzebeesSDK.instance().createAccessTokenRequest()

// Check availability before using shopping cart features
if (BuzzebeesSDK.instance().isShoppingCartAvailable()) {
    // All required config is present - safe to use cart functions
    val result = BuzzebeesSDK.instance().buildDefaultShoppingCartUrl()
}

// Validate a request before using it
val request = AccessTokenRequest(/*...*/) 
if (BuzzebeesSDK.instance().validateAccessTokenRequest(request)) {
    // Request is valid - safe to use
    val result = BuzzebeesSDK.instance().buildShoppingCartUrl(request)
}
```

---

## Usage Examples

### Basic Usage in Activities

```kotlin
class CampaignDetailActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_campaign_detail)
        
        val campaign = getCampaignFromIntent()
        val userId = getCurrentUserId()
        
        // Load campaign image
        val imageUrl = BuzzebeesSDK.instance().buildFullImageUrl(
            image = campaign.imageUrl,
            id = campaign.id
        )
        
        Glide.with(this)
            .load(imageUrl)
            .into(campaignImageView)
        
        // Set up redeem button
        redeemButton.setOnClickListener {
            val redeemUrl = BuzzebeesSDK.instance().buildRedeemUrl(
                campaign = campaign,
                userId = userId,
                options = getRedeemOptions()
            )
            
            // Open redemption URL
            startActivity(Intent(Intent.ACTION_VIEW, Uri.parse(redeemUrl)))
        }
        
        // Load website URL
        websiteButton.setOnClickListener {
            val websiteUrl = BuzzebeesSDK.instance().buildWebsiteUrl(campaign)
            startActivity(Intent(Intent.ACTION_VIEW, Uri.parse(websiteUrl)))
        }
    }
    
    private fun getRedeemOptions(): RedeemOptions {
        return RedeemOptions(
            color = colorSpinner.selectedItem.toString(),
            size = sizeSpinner.selectedItem.toString(),
            quantity = quantityPicker.value
        )
    }
}
```

### Integration with RecyclerView

```kotlin
class CampaignAdapter : RecyclerView.Adapter<CampaignViewHolder>() {
    
    override fun onBindViewHolder(holder: CampaignViewHolder, position: Int) {
        val campaign = campaigns[position]
        
        // Bind campaign image
        val imageUrl = BuzzebeesSDK.instance().buildFullImageUrl(
            image = campaign.imageUrl,
            id = campaign.id
        )
        
        Glide.with(holder.itemView.context)
            .load(imageUrl)
            .placeholder(R.drawable.campaign_placeholder)
            .error(R.drawable.campaign_error)
            .into(holder.campaignImage)
        
        // Set up click listener for redemption
        holder.redeemButton.setOnClickListener {
            val userId = UserManager.getCurrentUserId()
            val redeemUrl = BuzzebeesSDK.instance().buildRedeemUrl(
                campaign = campaign,
                userId = userId
            )
            
            val context = holder.itemView.context
            context.startActivity(Intent(Intent.ACTION_VIEW, Uri.parse(redeemUrl)))
        }
    }
}
```

### Profile Image Loading

```kotlin
class ProfileFragment : Fragment() {
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        val userId = arguments?.getString("user_id") ?: return
        
        // Load user profile image
        val profileImageUrl = BuzzebeesSDK.instance().buildProfileImageUrl(userId)
        
        Glide.with(this)
            .load(profileImageUrl)
            .circleCrop()
            .placeholder(R.drawable.default_profile)
            .error(R.drawable.profile_error)
            .into(profileImageView)
    }
}
```

### Shopping Cart URL Building

```kotlin
class CampaignDetailViewModel : ViewModel() {
    
    // Complete shopping cart flow with automatic SDK configuration
    fun openShoppingCart() {
        viewModelScope.launch {
            try {
                pageState.value = CampaignDetailPageState.Loading
                
                // Uses SDK configuration automatically:
                // - appId from configs.appId
                // - appName from configs.appName  
                // - returnUrl from configs.urlSchemesMainProject
                // - baseCartUrl from configs.webShoppingUrl
                val result = BuzzebeesSDK.instance().buildDefaultShoppingCartUrl()
                
                pageState.value = CampaignDetailPageState.Data
                
                when (result) {
                    is ShoppingCartUrlBuildResult.Success -> {
                        val accessToken = result.result.accessToken
                        
                        // Save access token
                        appPreference.saveShoppingCartAccessToken(accessToken)
                        
                        // Build specific cart URLs using the token
                        val cartUrl = BuzzebeesSDK.instance().buildFlexibleCartUrl(accessToken)
                        val myOrderUrl = BuzzebeesSDK.instance().buildFlexibleCartUrl(accessToken, "/myorder")
                        val voucherUrl = BuzzebeesSDK.instance().buildFlexibleCartUrl(accessToken, "/voucher")
                        
                        // Open shopping cart
                        event.setEventValue(
                            CampaignDetailEvent.OpenShoppingCart(cartUrl)
                        )
                    }
                    
                    is ShoppingCartUrlBuildResult.Error -> {
                        // Handle error
                        event.setEventValue(
                            CampaignDetailEvent.Failure(result.message.packMessage())
                        )
                    }
                }
            } catch (e: Exception) {
                pageState.value = CampaignDetailPageState.Data
                // Handle exception
            }
        }
    }
    
    // Manual configuration approach (if needed)
    fun openShoppingCartManual() {
        viewModelScope.launch {
            try {
                // Create custom AccessTokenRequest
                val request = BuzzebeesSDK.instance().createAccessTokenRequest(
                    customReturnUrl = "customapp://return",
                    appId = "custom-app-id", 
                    appName = "CustomAppName"
                )
                
                val result = BuzzebeesSDK.instance().buildShoppingCartUrl(request)
                
                when (result) {
                    is ShoppingCartUrlBuildResult.Success -> {
                        // Handle success
                        openShoppingCartUrl(result.result.url)
                    }
                    is ShoppingCartUrlBuildResult.Error -> {
                        showError(result.message)
                    }
                }
            } catch (e: Exception) {
                // Handle exception
            }
        }
    }
    
    // Navigate to specific cart pages with existing token
    fun navigateToMyOrders() {
        val accessToken = getStoredAccessToken() ?: return
        
        val myOrderUrl = BuzzebeesSDK.instance().buildFlexibleCartUrl(
            accessToken = accessToken,
            path = "/myorder",
            additionalParams = mapOf(
                "sortBy" to "date",
                "filter" to "active"
            )
        )
        
        openWebView(myOrderUrl)
    }
    
    fun navigateToVouchers(category: String? = null) {
        val accessToken = getStoredAccessToken() ?: return
        
        val additionalParams = category?.let { 
            mapOf("category" to it) 
        } ?: emptyMap()
        
        val voucherUrl = BuzzebeesSDK.instance().buildFlexibleCartUrl(
            accessToken = accessToken,
            path = "/voucher",
            additionalParams = additionalParams
        )
        
        openWebView(voucherUrl)
    }
    
    fun navigateToSettings() {
        val accessToken = getStoredAccessToken() ?: return
        
        val settingsUrl = BuzzebeesSDK.instance().buildFlexibleCartUrl(
            accessToken = accessToken,
            path = "/settings",
            additionalParams = mapOf("tab" to "profile")
        )
        
        openWebView(settingsUrl)
    }
}
```

---

## Integration Patterns

### Pattern 1: Repository Pattern

```kotlin
interface CampaignRepository {
    fun getCampaignImageUrl(campaign: Campaign): String
    fun getCampaignWebsiteUrl(campaign: Campaign): String
    fun generateRedeemUrl(campaign: Campaign, userId: String, options: RedeemOptions): String
}

class CampaignRepositoryImpl : CampaignRepository {
    
    override fun getCampaignImageUrl(campaign: Campaign): String {
        return BuzzebeesSDK.instance().buildFullImageUrl(
            image = campaign.imageUrl,
            id = campaign.id
        )
    }
    
    override fun getCampaignWebsiteUrl(campaign: Campaign): String {
        return BuzzebeesSDK.instance().buildWebsiteUrl(campaign)
    }
    
    override fun generateRedeemUrl(campaign: Campaign, userId: String, options: RedeemOptions): String {
        return BuzzebeesSDK.instance().buildRedeemUrl(
            campaign = campaign,
            userId = userId,
            options = options
        )
    }
}
```

### Pattern 2: Use Case Pattern

```kotlin
class GetCampaignUrlsUseCase {
    
    data class CampaignUrls(
        val imageUrl: String,
        val websiteUrl: String,
        val redeemUrl: String
    )
    
    operator fun invoke(
        campaign: Campaign,
        userId: String,
        redeemOptions: RedeemOptions = RedeemOptions()
    ): CampaignUrls {
        val sdk = BuzzebeesSDK.instance()
        
        return CampaignUrls(
            imageUrl = sdk.buildFullImageUrl(campaign.imageUrl, campaign.id),
            websiteUrl = sdk.buildWebsiteUrl(campaign),
            redeemUrl = sdk.buildRedeemUrl(campaign, userId, redeemOptions)
        )
    }
}

// Usage in ViewModel
class CampaignDetailViewModel : ViewModel() {
    private val getCampaignUrls = GetCampaignUrlsUseCase()
    
    fun loadCampaignUrls(campaign: Campaign, userId: String) {
        val urls = getCampaignUrls(campaign, userId)
        // Update UI state with URLs
    }
}
```

### Pattern 3: Extension Functions for Custom Models

```kotlin
// Extension functions on your own models
fun Campaign.getImageUrl(): String {
    return BuzzebeesSDK.instance().buildFullImageUrl(
        image = this.imageUrl,
        id = this.id
    )
}

fun Campaign.getWebsiteUrl(): String {
    return BuzzebeesSDK.instance().buildWebsiteUrl(this)
}

fun Campaign.getRedeemUrl(userId: String, options: RedeemOptions = RedeemOptions()): String {
    return BuzzebeesSDK.instance().buildRedeemUrl(
        campaign = this,
        userId = userId,
        options = options
    )
}

// Usage
val campaign = getCampaign()
val imageUrl = campaign.getImageUrl()
val redeemUrl = campaign.getRedeemUrl(userId, redeemOptions)
```

### Pattern 4: Shopping Cart Manager Pattern

```kotlin
class ShoppingCartManager {
    private var cachedAccessToken: String? = null
    private var tokenTimestamp: Long = 0
    private val tokenValidityDuration = TimeUnit.HOURS.toMillis(2) // 2 hours
    
    suspend fun initializeCartAccess(): CartAccessResult {
        return try {
            // Check if we have a valid cached token
            if (isTokenValid()) {
                CartAccessResult.Success(cachedAccessToken!!)
            } else {
                // Get new access token using SDK defaults
                val result = BuzzebeesSDK.instance().buildDefaultShoppingCartUrl()
                
                when (result) {
                    is ShoppingCartUrlBuildResult.Success -> {
                        cachedAccessToken = result.result.accessToken
                        tokenTimestamp = System.currentTimeMillis()
                        CartAccessResult.Success(result.result.accessToken)
                    }
                    is ShoppingCartUrlBuildResult.Error -> {
                        CartAccessResult.Error(result.message)
                    }
                }
            }
        } catch (e: Exception) {
            CartAccessResult.Error("Failed to initialize cart access: ${e.message}")
        }
    }
    
    private fun isTokenValid(): Boolean {
        return cachedAccessToken != null && 
               (System.currentTimeMillis() - tokenTimestamp) < tokenValidityDuration
    }
    
    suspend fun navigateToCart(): String? {
        return when (val result = initializeCartAccess()) {
            is CartAccessResult.Success -> {
                BuzzebeesSDK.instance().buildFlexibleCartUrl(result.accessToken)
            }
            is CartAccessResult.Error -> null
        }
    }
    
    suspend fun navigateToMyOrders(filters: Map<String, String> = emptyMap()): String? {
        return when (val result = initializeCartAccess()) {
            is CartAccessResult.Success -> {
                BuzzebeesSDK.instance().buildFlexibleCartUrl(
                    accessToken = result.accessToken,
                    path = "/myorder",
                    additionalParams = filters
                )
            }
            is CartAccessResult.Error -> null
        }
    }
    
    suspend fun navigateToVouchers(category: String? = null): String? {
        return when (val result = initializeCartAccess()) {
            is CartAccessResult.Success -> {
                val params = category?.let { mapOf("category" to it) } ?: emptyMap()
                BuzzebeesSDK.instance().buildFlexibleCartUrl(
                    accessToken = result.accessToken,
                    path = "/voucher",
                    additionalParams = params
                )
            }
            is CartAccessResult.Error -> null
        }
    }
    
    suspend fun navigateToCustomPage(path: String, params: Map<String, String> = emptyMap()): String? {
        return when (val result = initializeCartAccess()) {
            is CartAccessResult.Success -> {
                BuzzebeesSDK.instance().buildFlexibleCartUrl(
                    accessToken = result.accessToken,
                    path = path,
                    additionalParams = params
                )
            }
            is CartAccessResult.Error -> null
        }
    }
    
    fun clearCache() {
        cachedAccessToken = null
        tokenTimestamp = 0
    }
    
    sealed class CartAccessResult {
        data class Success(val accessToken: String) : CartAccessResult()
        data class Error(val message: String) : CartAccessResult()
    }
}

// Usage in ViewModel
class ShoppingViewModel : ViewModel() {
    private val cartManager = ShoppingCartManager()
    
    fun openCart() {
        viewModelScope.launch {
            val cartUrl = cartManager.navigateToCart()
            cartUrl?.let { openWebView(it) }
        }
    }
    
    fun openMyOrders() {
        viewModelScope.launch {
            val myOrderUrl = cartManager.navigateToMyOrders(
                filters = mapOf("status" to "active", "limit" to "20")
            )
            myOrderUrl?.let { openWebView(it) }
        }
    }
}
```

---

## Best Practices

### 1. Cache URLs When Possible

```kotlin
class CampaignUrlCache {
    private val imageUrlCache = mutableMapOf<String, String>()
    
    fun getCampaignImageUrl(campaign: Campaign): String {
        val cacheKey = "${campaign.id}_${campaign.imageUrl}"
        return imageUrlCache.getOrPut(cacheKey) {
            BuzzebeesSDK.instance().buildFullImageUrl(
                image = campaign.imageUrl,
                id = campaign.id
            )
        }
    }
}
```

### 2. Handle Null/Empty Campaigns Gracefully

```kotlin
fun Campaign?.getSafeImageUrl(): String {
    return if (this != null) {
        BuzzebeesSDK.instance().buildFullImageUrl(
            image = this.imageUrl,
            id = this.id
        )
    } else {
        "" // or default image URL
    }
}

fun Campaign?.getSafeRedeemUrl(userId: String?, options: RedeemOptions = RedeemOptions()): String? {
    return if (this != null && userId != null) {
        BuzzebeesSDK.instance().buildRedeemUrl(
            campaign = this,
            userId = userId,
            options = options
        )
    } else {
        null
    }
}
```

### 3. Use Default Options

```kotlin
object DefaultRedeemOptions {
    fun standard() = RedeemOptions(
        quantity = 1,
        color = "default",
        size = "medium"
    )
    
    fun premium(promoCode: String) = RedeemOptions(
        quantity = 1,
        customData = mapOf("promo" to promoCode, "tier" to "premium")
    )
}

// Usage
val redeemUrl = BuzzebeesSDK.instance().buildRedeemUrl(
    campaign = campaign,
    userId = userId,
    options = DefaultRedeemOptions.premium("SUMMER2024")
)
```

### 4. Error Handling

```kotlin
fun buildSafeRedeemUrl(
    campaign: Campaign?,
    userId: String?,
    options: RedeemOptions = RedeemOptions()
): String? {
    return try {
        if (campaign != null && userId != null) {
            BuzzebeesSDK.instance().buildRedeemUrl(campaign, userId, options)
        } else {
            null
        }
    } catch (e: Exception) {
        Log.e("URLBuilder", "Failed to build redeem URL", e)
        null
    }
}
```

### 5. Testing URL Generation

```kotlin
class CampaignUrlTest {
    
    @Test
    fun `test campaign image URL generation`() {
        // Given
        val campaign = Campaign(
            id = "test_123",
            imageUrl = "test-image.jpg"
        )
        
        // When
        val imageUrl = BuzzebeesSDK.instance().buildFullImageUrl(
            image = campaign.imageUrl,
            id = campaign.id
        )
        
        // Then
        assertTrue(imageUrl.contains("test-image.jpg"))
        assertTrue(imageUrl.startsWith("https://"))
    }
}
```

### 6. Shopping Cart Best Practices

```kotlin
// 1. Check availability before using cart features
fun initializeShoppingCart() {
    if (!BuzzebeesSDK.instance().isShoppingCartAvailable()) {
        // Handle missing configuration
        showError("Shopping cart is not available. Please check configuration.")
        return
    }
    
    // Proceed with cart initialization
}

// 2. Cache access tokens with expiration
class CartTokenManager {
    private val preferences = getSharedPreferences("cart_prefs", Context.MODE_PRIVATE)
    
    fun saveToken(token: String) {
        preferences.edit()
            .putString("access_token", token)
            .putLong("token_timestamp", System.currentTimeMillis())
            .apply()
    }
    
    fun getValidToken(): String? {
        val token = preferences.getString("access_token", null)
        val timestamp = preferences.getLong("token_timestamp", 0)
        val maxAge = TimeUnit.HOURS.toMillis(2) // 2 hours
        
        return if (token != null && (System.currentTimeMillis() - timestamp) < maxAge) {
            token
        } else {
            null
        }
    }
}

// 3. Handle network errors gracefully
suspend fun buildCartUrlSafely(): String? {
    return try {
        val result = BuzzebeesSDK.instance().buildDefaultShoppingCartUrl()
        when (result) {
            is ShoppingCartUrlBuildResult.Success -> {
                result.result.url
            }
            is ShoppingCartUrlBuildResult.Error -> {
                // Log error for debugging
                Timber.e("Cart URL build failed: ${result.message}")
                
                // Show user-friendly message
                when {
                    result.message.contains("network", ignoreCase = true) -> {
                        showError("Network error. Please check your connection.")
                    }
                    result.message.contains("configuration", ignoreCase = true) -> {
                        showError("Service temporarily unavailable.")
                    }
                    else -> {
                        showError("Unable to access shopping cart.")
                    }
                }
                null
            }
        }
    } catch (e: Exception) {
        Timber.e(e, "Exception building cart URL")
        showError("Unexpected error occurred.")
        null
    }
}

// 4. Validate configuration at app startup
class AppInitializer {
    fun validateSDKConfiguration() {
        val sdk = BuzzebeesSDK.instance()
        val configs = sdk.servicesConfigs()
        
        val issues = mutableListOf<String>()
        
        if (configs.webShoppingUrl.isNullOrEmpty()) {
            issues.add("webShoppingUrl not configured")
        }
        if (configs.appId.isNullOrEmpty()) {
            issues.add("appId not configured")
        }
        if (configs.appName.isNullOrEmpty()) {
            issues.add("appName not configured")
        }
        if (configs.urlSchemesMainProject.isNullOrEmpty()) {
            issues.add("urlSchemesMainProject not configured")
        }
        
        if (issues.isNotEmpty()) {
            Timber.w("SDK Configuration Issues: ${issues.joinToString(", ")}")
            // Consider disabling cart features
        }
    }
}
```

---

## Troubleshooting

### Common Issues and Solutions

#### 1. Empty or Null URLs

**Problem**: URL builder returns empty strings or null values

**Solution**: 
```kotlin
// Check SDK initialization
if (!BuzzebeesSDK.isInitialized()) {
    // Initialize SDK first
    BuzzebeesSDK.init(context)
}

// Verify campaign data
if (campaign?.imageUrl.isNullOrEmpty()) {
    // Use default image URL or handle gracefully
    return defaultImageUrl
}
```

#### 2. Authentication Token Missing

**Problem**: Profile image URLs don't include authentication tokens

**Solution**:
```kotlin
// Ensure user is authenticated
val authProvider = BuzzebeesSDK.instance().authProvider()
if (!authProvider.hasAuthToken()) {
    // Authenticate user first
    authenticateUser()
}

// Then build profile URL
val profileUrl = BuzzebeesSDK.instance().buildProfileImageUrl(userId)
```

#### 3. Incorrect Base URLs

**Problem**: Generated URLs use wrong base URLs

**Solution**:
```kotlin
// Check SDK configuration
val configs = BuzzebeesSDK.instance().servicesConfigs()
Log.d("URLBuilder", "Base URL: ${configs.baseUrl}")

// Verify buzzebees-service.json configuration
// Ensure correct environment configuration (staging/production)
```

#### 4. Shopping Cart Configuration Issues

**Problem**: Shopping cart URLs fail to build due to missing configuration

**Solution**:
```kotlin
// Check shopping cart availability first
if (!BuzzebeesSDK.instance().isShoppingCartAvailable()) {
    Log.e("ShoppingCart", "Missing required configuration:")
    val configs = BuzzebeesSDK.instance().servicesConfigs()
    if (configs.webShoppingUrl.isNullOrEmpty()) Log.e("ShoppingCart", "- webShoppingUrl")
    if (configs.appId.isNullOrEmpty()) Log.e("ShoppingCart", "- appId")
    if (configs.appName.isNullOrEmpty()) Log.e("ShoppingCart", "- appName")
    if (configs.urlSchemesMainProject.isNullOrEmpty()) Log.e("ShoppingCart", "- urlSchemesMainProject")
    
    // Handle missing configuration
    return
}

// Proceed with cart operations
```

#### 5. Redeem URL Parameters Missing

**Problem**: Redeem URLs don't include expected parameters

**Solution**:
```kotlin
// Explicitly set redeem options
val options = RedeemOptions(
    color = "red",
    size = "L",
    quantity = 2,
    customData = mapOf(
        "promo" to "DISCOUNT20",
        "source" to "mobile_app"
    )
)

val redeemUrl = BuzzebeesSDK.instance().buildRedeemUrl(
    campaign = campaign,
    userId = userId,
    options = options
)

Log.d("URLBuilder", "Redeem URL: $redeemUrl")
```

### Debug Logging

Enable debug logging to troubleshoot URL generation:

```kotlin
// Check what URLs are being generated
val imageUrl = BuzzebeesSDK.instance().buildFullImageUrl(image, id)
Log.d("URLBuilder", "Generated image URL: $imageUrl")

val redeemUrl = BuzzebeesSDK.instance().buildRedeemUrl(campaign, userId, options)
Log.d("URLBuilder", "Generated redeem URL: $redeemUrl")

// Check URL builder configuration
val urlBuilder = BuzzebeesSDK.instance().urlBuilder
Log.d("URLBuilder", "URL Builder configs: ${urlBuilder.configs}")

// Debug shopping cart configuration
val configs = BuzzebeesSDK.instance().servicesConfigs()
Log.d("ShoppingCart", "Cart Config - webShoppingUrl: ${configs.webShoppingUrl}")
Log.d("ShoppingCart", "Cart Config - appName: ${configs.appName}")
Log.d("ShoppingCart", "Cart Config - urlSchemesMainProject: ${configs.urlSchemesMainProject}")
```

---

## Conclusion

The URL Builder Extensions provide a powerful and convenient way to generate URLs for various Buzzebees services, including comprehensive shopping cart functionality. By following the patterns and best practices outlined in this guide, you can create robust URL generation that handles authentication, configuration, and parameter management automatically.

Key advantages:
- **Automatic Configuration**: No manual URL construction required
- **Type Safety**: Strongly typed parameters prevent errors
- **Authentication Handling**: Automatic token inclusion where needed
- **Template Processing**: Smart URL template replacement
- **Shopping Cart Integration**: Complete cart URL management with token handling
- **Easy Integration**: Simple extension functions on the main SDK

### 🆕 CampaignDetails Support

The URL Builder Extensions now support `CampaignDetails` objects for enhanced URL building:

#### Enhanced Website URLs
```kotlin
// Standard Campaign
val websiteUrl = BuzzebeesSDK.instance().buildWebsiteUrl(campaign)

// 🆕 CampaignDetails with automatic parameters
val campaignDetails = CampaignDetails(id = 123, website = "https://example.com", categoryID = 5, type = 1, agencyID = 10)
val enhancedWebsiteUrl = BuzzebeesSDK.instance().buildWebsiteUrl(campaignDetails)
// Returns: "https://example.com?campaign_id=123&category_id=5&campaign_type=1&agency_id=10"
```

#### Enhanced Redeem URLs
```kotlin
// Standard Campaign
val redeemUrl = BuzzebeesSDK.instance().buildRedeemUrl(campaign, userId, options)

// 🆕 CampaignDetails support
val campaignDetails = getCampaignDetails()
val enhancedRedeemUrl = BuzzebeesSDK.instance().buildRedeemUrl(campaignDetails, userId, options)
// Uses comprehensive CampaignDetails data with full URL building logic
```

#### Usage Examples with CampaignDetails
```kotlin
class CampaignDetailActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        val campaignDetails = getCampaignDetailsFromIntent()
        val userId = getCurrentUserId()
        
        // Enhanced website URL with automatic parameters
        websiteButton.setOnClickListener {
            val websiteUrl = BuzzebeesSDK.instance().buildWebsiteUrl(campaignDetails)
            startActivity(Intent(Intent.ACTION_VIEW, Uri.parse(websiteUrl)))
        }
        
        // Enhanced redeem URL with CampaignDetails
        redeemButton.setOnClickListener {
            val redeemUrl = BuzzebeesSDK.instance().buildRedeemUrl(
                campaignDetails = campaignDetails,
                userId = userId,
                options = getRedeemOptions()
            )
            startActivity(Intent(Intent.ACTION_VIEW, Uri.parse(redeemUrl)))
        }
    }
}
```

The CampaignDetails support provides automatic parameter inclusion for:
- Campaign ID (`campaign_id`)
- Category ID (`category_id`) 
- Campaign Type (`campaign_type`)
- Agency ID (`agency_id`)

For additional implementation details, refer to the `BuzzebeesSDK_UrlExtension.kt` source file.
