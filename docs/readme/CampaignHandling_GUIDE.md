# Campaign Handling & Validation Guide

This guide covers campaign redemption validation flow and error handling patterns for the Buzzebees SDK.

---

## Overview

The campaign handling system implements a validation process following the standard flow: **call detail → check canRedeem status → if ready → call redeem, else show appropriate UI**. This validation flow ensures data integrity and provides clear user feedback for all campaign states.

**SDK Enhancement**: The `getCampaignDetail()` method automatically calculates and includes validation status (`canRedeem`, `errorConditionMessage`) and button catalog (`campaignCatalog`). This eliminates the need for complex client-side validation logic.

### Standard Campaign Flow

All implementations should follow this sequence:

1. **Call `getCampaignDetail()`** - Get campaign information from server
2. **Check `canRedeem` status** - SDK automatically validates business rules  
3. **Handle canRedeem result**:
   - ✅ **`canRedeem = true`** → Enable redeem button → Call `redeem()` when clicked
   - ❌ **`canRedeem = false`** → Show appropriate UI based on `errorConditionMessage`
4. **Determine button type** - Use `campaignCatalog` to show correct UI

### SDK-Calculated Validation Fields

The following fields in `CampaignDetails` are automatically calculated by the SDK:

```kotlin
// Validation status
var canRedeem: Boolean?              // Campaign is ready for redemption
var errorConditionMessage: String?   // Error message if not ready

// Button catalog - determines which button type to show
var campaignCatalog: CampaignButtonCatalog  // Button type enum

// Normalized display data
var displayCampaignName: String
var displayCampaignDescription: String
var displayConditions: String
var displayFullImageUrl: String
var displayPictures: List<String>
var displayCampaignPoint: String
var displayVariants: List<VariantOption>?
var displaySubVariants: Map<String, List<SubVariantOption>>?
```

**SDK Auto-Validation**: The SDK automatically checks (in priority order):
1. User authentication status (not device login)
2. User points vs required points
3. Campaign expiration using server time
4. Available quantity (`qty > 0`)
5. Item count vs quantity limits (`itemCountSold < quantity`)
6. User eligibility and condition pass status
7. All condition alerts and business rules

---

## Campaign Types Reference

| Type Value | Constant Name                        | Description                  |
|------------|--------------------------------------|------------------------------|
| 0          | CAMPAIGN_TYPE_DRAW                   | Draw/Lottery campaign        |
| 1          | CAMPAIGN_TYPE_FREE                   | Free campaign                |
| 2          | CAMPAIGN_TYPE_DEAL                   | Deal campaign                |
| 3          | CAMPAIGN_TYPE_BUY                    | Shopping/Purchase campaign   |
| 4          | CAMPAIGN_TYPE_BID                    | Bid/Auction campaign         |
| 8          | CAMPAIGN_TYPE_INTERFACE              | Interface campaign           |
| 9          | CAMPAIGN_TYPE_EVENT                  | Event campaign               |
| 10         | CAMPAIGN_TYPE_MEDIA                  | Media campaign               |
| 16         | CAMPAIGN_TYPE_NEW                    | New campaign                 |
| 20         | CAMPAIGN_TYPE_DONATE                 | Donation campaign            |
| 33         | CAMPAIGN_TYPE_MARKETPLACE_PRIVILEGE  | Marketplace privilege        |

---

## Campaign Button Catalog

The SDK automatically determines the appropriate button type based on campaign configuration:

```kotlin
sealed class CampaignButtonCatalog {
    // Non-redeemable campaigns (Event, Media, News)
    object NoButton : CampaignButtonCatalog()
    
    // Standard redemption
    data class RedeemButton(val buttonName: String) : CampaignButtonCatalog()
    
    // BUY campaigns without variants
    data class ShoppingButton(
        val firstButtonName: String,
        val secondButtonName: String? = null
    ) : CampaignButtonCatalog()
    
    // BUY campaigns with variants
    data class ShoppingWithStyleButton(
        val firstButtonName: String,
        val secondButtonName: String? = null
    ) : CampaignButtonCatalog()
    
    // Campaigns requiring delivery address
    data class AddressButton(val buttonName: String) : CampaignButtonCatalog()
    
    // DONATE campaigns with quantity selection
    data class QuantityButton(val buttonName: String) : CampaignButtonCatalog()
}
```

### Button Type Determination Logic

| Campaign Type | Condition | Button Catalog |
|---------------|-----------|----------------|
| Event (9), Media (10), New (16) | - | `NoButton` |
| BUY (3) | Has variants | `ShoppingWithStyleButton` |
| BUY (3) | No variants | `ShoppingButton` |
| Marketplace Privilege (33) | - | `ShoppingButton` |
| Interface (8) | Survey type | `RedeemButton("Take Survey")` |
| Interface (8) | Other | `RedeemButton("Open")` |
| Draw (0) | Has delivery | `AddressButton("Draw")` |
| Draw (0) | No delivery | `RedeemButton("Draw")` |
| Donate (20) | - | `QuantityButton("Donate")` |
| Other (Free, Deal, Bid, Survey, Install, Register, etc.) | Has delivery | `AddressButton` |
| Other | pointType = "use" | `RedeemButton("Redeem")` |
| Other | pointType = "get" | `RedeemButton("Get Points")` |

---

## Validation Implementation

### Basic Validation with canRedeem

```kotlin
class CampaignValidator {
    
    fun validateCampaignForRedemption(campaignDetails: CampaignDetails): CampaignValidationResult {
        return if (campaignDetails.canRedeem == true) {
            CampaignValidationResult.Success
        } else {
            val error = mapConditionMessageToError(
                message = campaignDetails.errorConditionMessage ?: "Campaign not available"
            )
            CampaignValidationResult.Error(error)
        }
    }
    
    private fun mapConditionMessageToError(message: String): CampaignValidationError {
        return when {
            message.contains("Not Authenticated", ignoreCase = true) ->
                CampaignValidationError.DeviceLogin(message)
            message.contains("Insufficient points", ignoreCase = true) ->
                CampaignValidationError.InsufficientPoints(message)
            message.contains("expired", ignoreCase = true) ->
                CampaignValidationError.Expired(message)
            message.contains("sold out", ignoreCase = true) ->
                CampaignValidationError.SoldOut(message)
            message.contains("Max redemption", ignoreCase = true) ->
                CampaignValidationError.MaxPerPerson(message)
            message.contains("cool down", ignoreCase = true) ->
                CampaignValidationError.CoolDown(message)
            message.contains("not started", ignoreCase = true) ->
                CampaignValidationError.CampaignPending(message)
            message.contains("Sponsor only", ignoreCase = true) ->
                CampaignValidationError.SponsorOnly(message)
            message.contains("App version", ignoreCase = true) ->
                CampaignValidationError.VersionExpired(message)
            message.contains("terms and conditions", ignoreCase = true) ->
                CampaignValidationError.TermsViolation(message)
            else -> CampaignValidationError.CustomCondition(message)
        }
    }
}
```

---

## Condition Alert Reference

| Condition Alert ID | Message | Error Type |
|--------------------|---------|------------|
| 1 | "Campaign sold out" | `SOLD_OUT` |
| 2 | "Max redemption per person reached" | `MAX_PER_PERSON` |
| 3 | "Campaign in cool down period" | `COOL_DOWN` |
| 1403 | "Condition invalid" | `CONDITION_INVALID` |
| 1406 | "Sponsor only campaign" | `SPONSOR_ONLY` |
| 1409 | "Campaign expired" | `EXPIRED` |
| 1410 | "Campaign not started yet" | `CAMPAIGN_PENDING` |
| 1416 | "App version expired" | `VERSION_EXPIRED` |
| 1427 | "This privilege cannot be redeemed under the specified terms and conditions" | `TERMS_VIOLATION` |
| - | "Not Authenticated" | `DEVICE_LOGIN` |
| - | "Insufficient points. Required: X, Available: Y" | `INSUFFICIENT_POINTS` |

---

## Error Classes

```kotlin
sealed class CampaignValidationError(val message: String, val errorId: String) {
    data class DeviceLogin(val msg: String) : CampaignValidationError(msg, "DEVICE_LOGIN")
    data class InsufficientPoints(val msg: String) : CampaignValidationError(msg, "INSUFFICIENT_POINTS")
    data class Expired(val msg: String) : CampaignValidationError(msg, "EXPIRED")
    data class SoldOut(val msg: String) : CampaignValidationError(msg, "SOLD_OUT")
    data class MaxPerPerson(val msg: String) : CampaignValidationError(msg, "MAX_PER_PERSON")
    data class CoolDown(val msg: String) : CampaignValidationError(msg, "COOL_DOWN")
    data class ConditionInvalid(val msg: String) : CampaignValidationError(msg, "CONDITION_INVALID")
    data class SponsorOnly(val msg: String) : CampaignValidationError(msg, "SPONSOR_ONLY")
    data class CampaignPending(val msg: String) : CampaignValidationError(msg, "CAMPAIGN_PENDING")
    data class VersionExpired(val msg: String) : CampaignValidationError(msg, "VERSION_EXPIRED")
    data class TermsViolation(val msg: String) : CampaignValidationError(msg, "TERMS_VIOLATION")
    data class CustomCondition(val msg: String) : CampaignValidationError(msg, "CUSTOM_CONDITION")
}

sealed class CampaignValidationResult {
    object Success : CampaignValidationResult()
    data class Error(val error: CampaignValidationError) : CampaignValidationResult()
}
```

---

## Implementation Example

```kotlin
class CampaignDetailActivity : AppCompatActivity() {
    
    private val validator = CampaignValidator()
    private val campaignDetailUseCase = BuzzebeesSDK.instance().campaignDetailUseCase
    private lateinit var redeemButton: Button
    private lateinit var secondaryButton: Button
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_campaign_detail)
        
        redeemButton = findViewById(R.id.btn_redeem)
        secondaryButton = findViewById(R.id.btn_secondary)
        
        val campaignId = intent.getStringExtra("campaign_id") ?: return
        loadCampaignDetail(campaignId)
    }
    
    private fun loadCampaignDetail(campaignId: String) {
        campaignDetailUseCase.getCampaignDetail(campaignId) { result ->
            when (result) {
                is CampaignDetailResult.SuccessCampaignDetail -> {
                    val campaignDetails = result.result
                    displayCampaignInfo(campaignDetails)
                    setupButtonCatalog(campaignDetails)
                    handleValidation(campaignDetails)
                }
                is CampaignDetailResult.Error -> {
                    showError("Failed to load campaign: ${result.error.message}")
                }
            }
        }
    }
    
    private fun setupButtonCatalog(campaignDetails: CampaignDetails) {
        when (val catalog = campaignDetails.campaignCatalog) {
            is CampaignButtonCatalog.NoButton -> {
                redeemButton.visibility = View.GONE
                secondaryButton.visibility = View.GONE
            }
            is CampaignButtonCatalog.RedeemButton -> {
                redeemButton.text = catalog.buttonName
                redeemButton.visibility = View.VISIBLE
                secondaryButton.visibility = View.GONE
            }
            is CampaignButtonCatalog.ShoppingButton -> {
                redeemButton.text = catalog.firstButtonName
                redeemButton.visibility = View.VISIBLE
                catalog.secondButtonName?.let {
                    secondaryButton.text = it
                    secondaryButton.visibility = View.VISIBLE
                }
            }
            is CampaignButtonCatalog.ShoppingWithStyleButton -> {
                // Show variant selector first
                showVariantSelector(campaignDetails.displayVariants ?: emptyList())
                redeemButton.text = catalog.firstButtonName
                catalog.secondButtonName?.let {
                    secondaryButton.text = it
                    secondaryButton.visibility = View.VISIBLE
                }
            }
            is CampaignButtonCatalog.AddressButton -> {
                redeemButton.text = catalog.buttonName
                redeemButton.visibility = View.VISIBLE
                // Show address selector
                showAddressSelector()
            }
            is CampaignButtonCatalog.QuantityButton -> {
                redeemButton.text = catalog.buttonName
                redeemButton.visibility = View.VISIBLE
                // Show quantity selector
                showQuantitySelector()
            }
        }
    }
    
    private fun handleValidation(campaignDetails: CampaignDetails) {
        val validationResult = validator.validateCampaignForRedemption(campaignDetails)
        
        when (validationResult) {
            is CampaignValidationResult.Success -> {
                redeemButton.isEnabled = true
                redeemButton.setOnClickListener {
                    performRedemption()
                }
            }
            is CampaignValidationResult.Error -> {
                handleValidationError(validationResult.error, campaignDetails)
            }
        }
    }
    
    private fun handleValidationError(error: CampaignValidationError, campaignDetails: CampaignDetails) {
        redeemButton.isEnabled = false
        
        when (error) {
            is CampaignValidationError.DeviceLogin -> {
                redeemButton.visibility = View.GONE
                showMessage("Please login to redeem campaigns")
            }
            is CampaignValidationError.InsufficientPoints -> {
                redeemButton.text = "Insufficient Points"
                showPointsDialog(campaignDetails.pointPerUnit ?: 0.0)
            }
            is CampaignValidationError.SoldOut -> {
                redeemButton.text = "Sold Out"
            }
            is CampaignValidationError.Expired -> {
                redeemButton.text = "Expired"
            }
            is CampaignValidationError.VersionExpired -> {
                showUpdateDialog()
            }
            is CampaignValidationError.MaxPerPerson -> {
                redeemButton.text = "Limit Reached"
            }
            is CampaignValidationError.CoolDown -> {
                redeemButton.text = "Cool Down"
                showCoolDownInfo(campaignDetails.nextRedeemDate)
            }
            is CampaignValidationError.CampaignPending -> {
                redeemButton.text = "Not Started"
                showStartDateInfo(campaignDetails.startDate)
            }
            is CampaignValidationError.SponsorOnly -> {
                redeemButton.text = "Sponsor Only"
                showSponsorDialog()
            }
            else -> {
                redeemButton.text = "Not Available"
                showMessage(error.message)
            }
        }
    }
    
    private fun performRedemption() {
        campaignDetailUseCase.redeem(mapOf()) { result ->
            when (result) {
                is CampaignDetailResult.SuccessRedeem -> {
                    handleRedemptionSuccess(result.result, result.nextStep)
                }
                is CampaignDetailResult.Error -> {
                    showError(result.error.message ?: "Redemption failed")
                }
            }
        }
    }
    
    private fun handleRedemptionSuccess(response: RedeemResponse, nextStep: NextStep) {
        when (nextStep) {
            is NextStep.ShowCode -> {
                showRedemptionCodeDialog(nextStep.code, nextStep.redeemKey)
            }
            is NextStep.ShowPointsEarned -> {
                showPointsEarnedDialog(nextStep.pointsEarned)
            }
            is NextStep.ShowDrawSuccess -> {
                showDrawSuccessDialog(nextStep.code, nextStep.pointsEarned)
            }
            is NextStep.ShowAddToCartSuccess -> {
                showAddToCartSuccessDialog(nextStep.cartUrl)
            }
            is NextStep.OpenWebsite -> {
                openWebView(nextStep.url, nextStep.urlType)
            }
        }
    }
}
```

---

## Variant Selection Flow (BUY Campaigns)

For BUY campaigns with variants (`ShoppingWithStyleButton`), use the variant selection methods:

```kotlin
class BuyCampaignFlow {
    private val campaignDetailUseCase = BuzzebeesSDK.instance().campaignDetailUseCase
    
    fun setupVariantSelection(campaignDetails: CampaignDetails) {
        val variants = campaignDetails.displayVariants ?: return
        
        // Display variant options (e.g., colors)
        variants.forEach { variant ->
            val button = createVariantButton(variant)
            button.setOnClickListener {
                onVariantSelected(variant, campaignDetails)
            }
        }
    }
    
    private fun onVariantSelected(variant: VariantOption, campaignDetails: CampaignDetails) {
        // Select variant in SDK
        val error = campaignDetailUseCase.selectVariant(variant)
        if (error != null) {
            showError(error)
            return
        }
        
        // Check for sub-variants
        val subVariants = campaignDetails.displaySubVariants?.get(variant.campaignId)
        if (!subVariants.isNullOrEmpty()) {
            showSubVariantSelector(subVariants)
        } else {
            enableRedeemButton()
        }
    }
    
    private fun onSubVariantSelected(subVariant: SubVariantOption) {
        val error = campaignDetailUseCase.selectSubVariant(subVariant)
        if (error != null) {
            showError(error)
            return
        }
        enableRedeemButton()
    }
    
    private fun onQuantityChanged(quantity: Int) {
        val error = campaignDetailUseCase.setQuantity(quantity)
        if (error != null) {
            showError(error)
        }
    }
    
    private fun addToCart() {
        campaignDetailUseCase.redeem(mapOf()) { result ->
            when (result) {
                is CampaignDetailResult.SuccessRedeem -> {
                    if (result.nextStep is NextStep.ShowAddToCartSuccess) {
                        showCartSuccess(result.nextStep.cartUrl)
                    }
                }
                is CampaignDetailResult.Error -> {
                    showError(result.error.message ?: "Failed to add to cart")
                }
            }
        }
    }
}
```

### VariantOption Fields

| Field | Description | Data Type |
|-------|-------------|-----------|
| campaignId | Campaign ID for this variant | String? |
| name | Display name | String? |
| value | Value | String? |
| price | Price | Double? |
| points | Points required | Double? |
| quantity | Available quantity | Int? |

### SubVariantOption Fields

| Field | Description | Data Type |
|-------|-------------|-----------|
| campaignId | Campaign ID for this sub-variant | String? |
| name | Display name | String? |
| value | Value | String? |
| type | Sub-variant type | String? |
| price | Price | Double? |
| points | Points required | Double? |
| quantity | Available quantity | Int? |

---

## Address Selection Flow (Delivery Campaigns)

For campaigns requiring delivery (`AddressButton`):

```kotlin
class DeliveryCampaignFlow {
    private val campaignDetailUseCase = BuzzebeesSDK.instance().campaignDetailUseCase
    
    fun onAddressSelected(address: Address) {
        val error = campaignDetailUseCase.selectAddress(address)
        if (error != null) {
            showError(error)
            return
        }
        enableRedeemButton()
    }
    
    fun redeem() {
        campaignDetailUseCase.redeem(mapOf()) { result ->
            when (result) {
                is CampaignDetailResult.SuccessRedeem -> {
                    handleSuccess(result.nextStep)
                }
                is CampaignDetailResult.Error -> {
                    showError(result.error.message ?: "Redemption failed")
                }
            }
        }
    }
}
```

---

## Quantity Selection Flow (DONATE Campaigns)

For DONATE campaigns (`QuantityButton`):

```kotlin
class DonateCampaignFlow {
    private val campaignDetailUseCase = BuzzebeesSDK.instance().campaignDetailUseCase
    
    fun onIncreaseQuantity() {
        val error = campaignDetailUseCase.increaseQuantity()
        error?.let { showError(it) }
        updateQuantityDisplay(campaignDetailUseCase.getSelectedQuantity())
    }
    
    fun onDecreaseQuantity() {
        campaignDetailUseCase.decreaseQuantity()
        updateQuantityDisplay(campaignDetailUseCase.getSelectedQuantity())
    }
    
    fun donate() {
        campaignDetailUseCase.redeem(mapOf()) { result ->
            when (result) {
                is CampaignDetailResult.SuccessRedeem -> {
                    if (result.nextStep is NextStep.ShowDrawSuccess) {
                        showDonateSuccess()
                    }
                }
                is CampaignDetailResult.Error -> {
                    showError(result.error.message ?: "Donation failed")
                }
            }
        }
    }
}
```

---

## NextStep - Post-Redemption Flow

The SDK returns a `NextStep` indicating what UI action to take after redemption:

```kotlin
sealed class NextStep {
    // Display redemption code to user
    data class ShowCode(
        val redeemKey: String,
        val code: String,
        val campaignId: String
    ) : NextStep()
    
    // Display points earned
    data class ShowPointsEarned(
        val redeemKey: String,
        val pointsEarned: Double,
        val campaignId: String
    ) : NextStep()
    
    // Display draw/donate success
    data class ShowDrawSuccess(
        val redeemKey: String,
        val campaignId: String,
        val code: String? = null,
        val pointsEarned: Double? = null
    ) : NextStep()
    
    // Show cart success (BUY campaigns)
    data class ShowAddToCartSuccess(
        val cartUrl: String? = null
    ) : NextStep()
    
    // Open external URL (Interface, Media, News)
    data class OpenWebsite(
        val url: String,
        val urlType: String  // "cart", "survey", "website", "media", "marketplace"
    ) : NextStep()
}
```

---

## Complete Campaign Flow Summary

```kotlin
class CampaignFlowManager {
    private val campaignDetailUseCase = BuzzebeesSDK.instance().campaignDetailUseCase
    
    fun executeCampaignFlow(campaignId: String, callback: (CampaignFlowResult) -> Unit) {
        // Step 1: Get campaign details
        campaignDetailUseCase.getCampaignDetail(campaignId) { result ->
            when (result) {
                is CampaignDetailResult.SuccessCampaignDetail -> {
                    val details = result.result
                    
                    // Step 2: Check canRedeem
                    if (details.canRedeem != true) {
                        callback(CampaignFlowResult.ValidationFailed(
                            details.errorConditionMessage ?: "Not available"
                        ))
                        return@getCampaignDetail
                    }
                    
                    // Step 3: Determine UI based on campaignCatalog
                    callback(CampaignFlowResult.ReadyToRedeem(
                        details = details,
                        buttonCatalog = details.campaignCatalog
                    ))
                }
                is CampaignDetailResult.Error -> {
                    callback(CampaignFlowResult.LoadFailed(result.error.message ?: "Load failed"))
                }
            }
        }
    }
}

sealed class CampaignFlowResult {
    data class ReadyToRedeem(
        val details: CampaignDetails,
        val buttonCatalog: CampaignButtonCatalog
    ) : CampaignFlowResult()
    
    data class ValidationFailed(val message: String) : CampaignFlowResult()
    data class LoadFailed(val message: String) : CampaignFlowResult()
}
```

---

## Summary

This Campaign Handling & Validation Guide provides comprehensive SDK features for proper campaign handling:

### canRedeem - Campaign Validation

Automatic validation of campaign conditions (calculated by SDK):
- Authentication check (not device login)
- Points validation (user points vs required)
- Expiration validation (using server time)
- Inventory management (qty, itemCountSold)
- User eligibility (isConditionPass)
- Business rule enforcement (conditionAlertId)

### campaignCatalog - Button Type Determination

Pre-calculated button type based on campaign configuration:
- `NoButton` - Non-redeemable (Event, Media, News)
- `RedeemButton` - Standard redemption
- `ShoppingButton` - BUY without variants
- `ShoppingWithStyleButton` - BUY with variants
- `AddressButton` - Requires delivery address
- `QuantityButton` - DONATE with quantity

### Variant/Address/Quantity Selection

SDK methods for managing selections:
- `selectVariant()` / `selectSubVariant()` - BUY campaigns
- `selectAddress()` - Delivery campaigns
- `setQuantity()` / `increaseQuantity()` / `decreaseQuantity()` - Quantity selection
- `clearAllSelections()` - Reset all selections

### NextStep - Post-Redemption Flow

Clear guidance on what UI to show after redemption:
- `ShowCode` - Display redemption code
- `ShowPointsEarned` - Display points earned
- `ShowDrawSuccess` - Draw/donate success
- `ShowAddToCartSuccess` - Cart success
- `OpenWebsite` - Open webview

### Key Features

- **SDK Auto-Validation**: Use built-in `canRedeem`, `errorConditionMessage`, `campaignCatalog`
- **Simple Integration**: Clear patterns for all campaign types
- **Server-Side Calculation**: Secure, prevents client manipulation
- **Type-Safe**: Sealed classes, no magic strings
- **Unified Flow**: Single `redeem()` method handles all campaign types

---

## Related Documentation

- [CampaignDetailUseCase_GUIDE.md](./CampaignDetailUseCase_GUIDE.md) - Detailed API reference
- [CampaignUseCase_GUIDE.md](./CampaignUseCase_GUIDE.md) - Campaign list operations
