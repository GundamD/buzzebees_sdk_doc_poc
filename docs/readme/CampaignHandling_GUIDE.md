# Campaign Handling & Validation Guide

This guide covers campaign redemption validation flow and error handling patterns for the Buzzebees SDK.

---

## Overview

The campaign handling system implements a validation process following the standard flow: **call detail → check canRedeem status → if ready → call redeem, else show appropriate UI**. This validation flow ensures data integrity and provides clear user feedback for all campaign states.

**SDK Enhancement**: The `getCampaignDetail()` method automatically calculates and includes validation status (`canRedeem`, `errorConditionMessage`) and button catalog (`campaignCatalog`). This eliminates the need for complex client-side validation logic.

**Localization Support**: Use `setDisplayTexts()` to configure all button names, error messages, and condition alerts in your preferred language. See [setDisplayTexts Configuration](#setdisplaytexts-configuration) section.

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
| 16         | CAMPAIGN_TYPE_NEWS                   | News campaign                |
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
| Other | Has delivery | `AddressButton` |
| Other | pointType = "use" | `RedeemButton("Redeem")` |
| Other | pointType = "get" | `RedeemButton("Get Points")` |

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
| 1427 | "Terms and conditions violation" | `TERMS_VIOLATION` |

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

## NextStep - Post-Redemption Flow

The SDK returns a `NextStep` indicating what UI action to take after redemption:

```kotlin
sealed class NextStep {
    data class ShowCode(val redeemKey: String, val code: String, val campaignId: String) : NextStep()
    data class ShowPointsEarned(val redeemKey: String, val pointsEarned: Double, val campaignId: String) : NextStep()
    data class ShowDrawSuccess(val redeemKey: String, val campaignId: String, val code: String? = null, val pointsEarned: Double? = null) : NextStep()
    data class ShowAddToCartSuccess(val cartUrl: String? = null) : NextStep()
    data class OpenWebsite(val url: String, val urlType: String) : NextStep()
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

---

## setDisplayTexts Configuration

Configure all button names, error messages, and condition alerts once at initialization:

```kotlin
BuzzebeesSDK.instance().campaignDetailUseCase.setDisplayTexts(
    CampaignDetailExtractorConfig.THAI
)
```

### Available Presets

```kotlin
CampaignDetailExtractorConfig.DEFAULT  // English
CampaignDetailExtractorConfig.THAI     // Thai
```

For complete field reference, see [CampaignDetailUseCase Guide](./CampaignDetailUseCase_GUIDE.md#setdisplaytexts).

---

## Related Documentation

- [SDK Comprehensive Guide](./SDK_COMPREHENSIVE_GUIDE.md) - Complete overview of all SDK capabilities
- [CampaignDetailUseCase Guide](./CampaignDetailUseCase_GUIDE.md) - Campaign detail and redemption API
- [CampaignUseCase Guide](./CampaignUseCase_GUIDE.md) - Campaign list operations
