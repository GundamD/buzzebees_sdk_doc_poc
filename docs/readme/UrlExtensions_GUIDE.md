# URL Builder Extensions Guide

This guide provides comprehensive documentation for using the BuzzebeesSDK URL building capabilities through the `BuzzebeesSDK_UrlExtension.kt` file. These extensions allow you to build URLs for campaigns, profile images, and redemption flows with automatic configuration and parameter handling.

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

The extension functions provide direct access without needing to manage the URL builder instance:

```kotlin
val profileUrl = BuzzebeesSDK.instance().buildProfileImageUrl(userId)
val campaignImageUrl = BuzzebeesSDK.instance().buildFullImageUrl(image, campaignId)
val websiteUrl = BuzzebeesSDK.instance().buildWebsiteUrl(campaign)
val redeemUrl = BuzzebeesSDK.instance().buildRedeemUrl(campaign, userId, options)
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

#### 4. Redeem URL Parameters Missing

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
```

---

## Conclusion

The URL Builder Extensions provide a powerful and convenient way to generate URLs for various Buzzebees services. By following the patterns and best practices outlined in this guide, you can create robust URL generation that handles authentication, configuration, and parameter management automatically.

Key advantages:
- **Automatic Configuration**: No manual URL construction required
- **Type Safety**: Strongly typed parameters prevent errors
- **Authentication Handling**: Automatic token inclusion where needed
- **Template Processing**: Smart URL template replacement
- **Easy Integration**: Simple extension functions on the main SDK

For additional implementation details, refer to the `BuzzebeesSDK_UrlExtension.kt` source file and the `BuzzebeesUrlBuilder` utility class.
