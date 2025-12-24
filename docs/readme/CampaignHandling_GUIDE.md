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
| Other (Free, Deal, Bid, etc.) | Has delivery | `AddressButton` |
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
| 1427 | "Terms and conditions not met" | `TERMS_VIOLATION` |
| - | "Not Authenticated" | `DEVICE_LOGIN` |
| - | "Insufficient points" | `INSUFFICIENT_POINTS` |

---

## NextStep - Post-Redemption Flow

The SDK returns a `NextStep` indicating what UI action to take after redemption:

```kotlin
sealed class NextStep {
    data class ShowCode(val redeemKey: String, val code: String, val campaignId: String) : NextStep()
    data class ShowPointsEarned(val redeemKey: String, val pointsEarned: Double, val campaignId: String) : NextStep()
    data class ShowDrawSuccess(val redeemKey: String, val campaignId: String, val code: String?, val pointsEarned: Double?) : NextStep()
    data class ShowAddToCartSuccess(val cartUrl: String?) : NextStep()
    data class OpenWebsite(val url: String, val urlType: String) : NextStep()
}
```

---

## Summary

### Key Features

- **SDK Auto-Validation**: Use built-in `canRedeem`, `errorConditionMessage`, `campaignCatalog`
- **Simple Integration**: Clear patterns for all campaign types
- **Server-Side Calculation**: Secure, prevents client manipulation
- **Type-Safe**: Sealed classes, no magic strings
- **Unified Flow**: Single `redeem()` method handles all campaign types

---

## setDisplayTexts Configuration

Configure all button names, error messages, and condition alerts once at initialization:

```kotlin
// Set display texts once at app startup
BuzzebeesSDK.instance().campaignDetailUseCase.setDisplayTexts(
    CampaignDetailExtractorConfig.THAI
)
```

### Available Presets

```kotlin
// English (Default)
CampaignDetailExtractorConfig.DEFAULT

// Thai
CampaignDetailExtractorConfig.THAI
```

### Configurable Fields

| Category | Fields |
|----------|--------|
| **Button Names** | buttonShopNow, buttonAddToCart, buttonTakeSurvey, buttonOpen, buttonDraw, buttonDonate, buttonRedeem, buttonGetPoints |
| **Condition Alerts** | alertSoldOut, alertMaxRedemption, alertCoolDown, alertConditionInvalid, alertSponsorOnly, alertExpired, alertNotStarted, alertAppVersionExpired, alertTermsConditions, alertUnknown |
| **Validation Errors** | errorNotAuthenticated, errorInsufficientPoints, errorCampaignExpired, errorCampaignSoldOut, errorCampaignNotLoaded, errorVariantOutOfStock, errorSubVariantOutOfStock, errorSelectVariantFirst, errorQuantityMinimum, errorOnlyXAvailable, errorMaxDonateAllowed, errorSelectAddress, errorSelectVariant, errorSelectSubVariant, errorTokenRequired, errorAddToCartFailed |

### Custom Configuration

```kotlin
val customConfig = CampaignDetailExtractorConfig.THAI.copy(
    buttonRedeem = "แลกสิทธิ์",
    buttonShopNow = "ช้อปเลย",
    alertSoldOut = "สินค้าหมดแล้วจ้า"
)
BuzzebeesSDK.instance().campaignDetailUseCase.setDisplayTexts(customConfig)
```

### When to Call

- **At app startup** - Set once in Application class
- **On language change** - Update when user changes app language

For complete field reference, see [CampaignDetailUseCase Guide](./CampaignDetailUseCase_GUIDE.md#setdisplaytexts).

---

## Related Documentation

- [CampaignDetailUseCase_GUIDE.md](./CampaignDetailUseCase_GUIDE.md) - Detailed API reference
- [CampaignUseCase_GUIDE.md](./CampaignUseCase_GUIDE.md) - Campaign list operations
