# Campaign Handling & Validation Guide

This comprehensive guide covers campaign redemption validation flow and error handling patterns for the Buzzebees SDK. It extends the **CampaignUseCase** with production-grade validation logic based on real application business rules.

---

## Overview

The campaign handling system implements a robust **validation process** following the standard flow: **call detail → validate readyToUse status → if ready → call redeem, else show alert or disable button**. This validation flow ensures data integrity, prevents fraudulent activities, and provides clear user feedback for all campaign states and error conditions.

**New SDK Enhancement**: Starting from the updated SDK version, `getCampaignDetail()` automatically calculates and includes a `readyToUse` status object that pre-validates campaign conditions. This eliminates the need for complex client-side validation logic while maintaining all validation functionality.

**Key Purpose**: Show developers how to properly implement the campaign detail validation flow that matches production Buzzebees application behavior.

### Standard Campaign Flow

The recommended implementation follows this sequence:

1. **Call `getCampaignDetail()`** - Get campaign information from server
2. **Check `readyToUse` status** - SDK automatically validates all business rules and user eligibility  
3. **Handle readyToUse result**:
   - ✅ **`isReadyToUse = true`** → Enable redeem button → Call `redeem()` when clicked
   - ❌ **`isReadyToUse = false`** → Show appropriate alert based on `message` and `code` or disable redeem button

### SDK-Calculated ReadyToUse

The `readyToUse` field in `CampaignDetails` is automatically calculated by the SDK and includes:

```kotlin
data class ReadyToUse(
    var isReadyToUse: Boolean? = false,  // Campaign is ready for redemption
    var message: String? = null,         // Status message (reason if not ready)
    var code: String? = null             // Status code identifier
)
```

**SDK Auto-Validation**: The SDK automatically checks:
- User authentication status (Device vs Authenticated login)
- Campaign expiration using server time
- Available quantity and sold count
- User eligibility and condition pass status
- All condition alerts and business rules

### Required Understanding

To effectively implement this flow, developers need to understand these related components:

- **[CampaignUseCase Guide](CampaignUseCase_GUIDE.md)** - Core campaign methods (`getCampaignDetail`, `redeem`)
- **[ProfileUseCase Guide](ProfileUseCase_GUIDE.md)** - User profile data (`loginType`, `currentPoints`, user authentication status)
- **Campaign Business Logic** - Understanding of `isConditionPass`, `conditionAlertId`, and validation priority

**Why ProfileUseCase Matters**: Campaign validation requires user profile information to determine login type (Device vs Authenticated), current point balance, and user eligibility. The validation logic heavily depends on these profile attributes.

This guide focuses specifically on **how to use the SDK's readyToUse validation** and **error handling patterns** for optimal user experience.

---

## SDK ReadyToUse Validation Approach

**Recommended Approach**: Use the SDK's built-in `readyToUse` validation for simplicity and consistency.

```kotlin
/**
 * Modern SDK approach - Use readyToUse field
 * Automatically calculated by SDK in getCampaignDetail()
 */
class ModernCampaignValidator {
    
    fun validateCampaignForRedemption(campaignDetails: CampaignDetails): CampaignValidationResult {
        val readyToUse = campaignDetails.readyToUse
        
        return if (readyToUse.isReadyToUse == true) {
            CampaignValidationResult.Success
        } else {
            // Convert readyToUse code to appropriate error
            val error = mapReadyToUseCodeToError(
                code = readyToUse.code,
                message = readyToUse.message ?: "Campaign not available"
            )
            CampaignValidationResult.Error(error)
        }
    }
    
    private fun mapReadyToUseCodeToError(code: String?, message: String): CampaignValidationError {
        return when (code) {
            "1" -> CampaignValidationError.SoldOut(message)
            "2" -> CampaignValidationError.MaxPerPerson(message)
            "3" -> CampaignValidationError.CoolDown(message)
            "1403" -> CampaignValidationError.ConditionInvalid(message)
            "1406" -> CampaignValidationError.SponsorOnly(message)
            "1409" -> CampaignValidationError.Expired(message)
            "1410" -> CampaignValidationError.CampaignPending(message)
            "1416" -> CampaignValidationError.VersionExpired(message)
            "1427" -> CampaignValidationError.TermsViolation(message)
            else -> {
                // Handle authentication and other conditions
                when {
                    message.contains("authenticated", ignoreCase = true) ->
                        CampaignValidationError.DeviceLogin(message)
                    message.contains("point", ignoreCase = true) ||
                    message.contains("insufficient", ignoreCase = true) ->
                        CampaignValidationError.InsufficientPoints(message)
                    else -> CampaignValidationError.CustomCondition(message)
                }
            }
        }
    }
}
```

## Legacy Manual Validation (Optional Reference)

**Note**: The following manual validation approach is maintained for reference and advanced use cases. **Most developers should use the SDK's readyToUse field** instead of implementing manual validation.

Based on actual implementation logic, the campaign validation follows this priority order:

```kotlin
/**
 * Production-grade campaign validation logic
 * Based on real application implementation patterns
 */
class ProductionCampaignValidator {
    
    fun validateCampaignForRedemption(
        campaignDetails: CampaignDetails,
        internalProfile: InternalProfile?,
        currentPoints: Double
    ): CampaignValidationResult {
        
        // Step 1: Check login type - Device login cannot redeem
        if (LoginType.fromValue(internalProfile?.loginType) == LoginType.DEVICE) {
            return CampaignValidationResult.Error(
                CampaignValidationError.DeviceLogin("Device users cannot redeem campaigns")
            )
        }
        
        // Step 2: Check point sufficiency
        val requiredPoints = campaignDetails.pointPerUnit ?: 0.0
        if (currentPoints < requiredPoints) {
            return CampaignValidationResult.Error(
                CampaignValidationError.InsufficientPoints("Point not enough")
            )
        }
        
        // Step 3: Check condition pass status
        if (campaignDetails.isConditionPass == true) {
            // 3a: Check expiry
            val remainingDays = calculateRemainingDays(campaignDetails)
            if (remainingDays < 1) {
                return CampaignValidationResult.Error(
                    CampaignValidationError.Expired("Campaign expired")
                )
            }
            
            // 3b: Check quantity availability
            val qty = campaignDetails.qty ?: 0.0
            if (qty.toInt() <= 0) {
                return CampaignValidationResult.Error(
                    CampaignValidationError.SoldOut("Campaign sold out")
                )
            }
            
            // 3c: Check item count vs quantity
            val itemCountSold = campaignDetails.itemCountSold
            val quantity = campaignDetails.quantity
            if (itemCountSold != null && quantity != null && 
                itemCountSold >= quantity) {
                return CampaignValidationResult.Error(
                    CampaignValidationError.SoldOut("Campaign sold out")
                )
            }
            
            // All checks passed
            return CampaignValidationResult.Success
        } else {
            // Step 4: Handle condition alerts when isConditionPass = false
            return handleConditionAlert(campaignDetails)
        }
    }
    
    private fun handleConditionAlert(details: CampaignDetails): CampaignValidationResult {
        return when (details.conditionAlertId?.toString()) {
            "1" -> CampaignValidationResult.Error(
                CampaignValidationError.SoldOut("Campaign sold out")
            )
            "2" -> CampaignValidationResult.Error(
                CampaignValidationError.MaxPerPerson("Max redemption per person reached")
            )
            "3" -> CampaignValidationResult.Error(
                CampaignValidationError.CoolDown("Campaign in cool down period")
            )
            "1403" -> CampaignValidationResult.Error(
                CampaignValidationError.ConditionInvalid("Condition invalid")
            )
            "1406" -> CampaignValidationResult.Error(
                CampaignValidationError.SponsorOnly("Sponsor only campaign")
            )
            "1409" -> CampaignValidationResult.Error(
                CampaignValidationError.Expired("Campaign expired")
            )
            "1410" -> CampaignValidationResult.Error(
                CampaignValidationError.CampaignPending("Campaign not started yet")
            )
            "1416" -> CampaignValidationResult.Error(
                CampaignValidationError.VersionExpired("App version expired")
            )
            "1427" -> CampaignValidationResult.Error(
                CampaignValidationError.TermsViolation("This privilege cannot be redeemed under the specified terms and conditions")
            )
            else -> {
                // Handle custom messages from server
                val customMessage = when {
                    !details.customCaption.isNullOrBlank() -> details.customCaption
                    !details.conditionAlert.isNullOrBlank() -> details.conditionAlert
                    else -> "Unknown condition error"
                }
                CampaignValidationResult.Error(
                    CampaignValidationError.CustomCondition(customMessage)
                )
            }
        }
    }
    
    /**
     * Calculate remaining days using server time to prevent client-side manipulation
     * Uses campaignDetails.currentDate (server time) instead of System.currentTimeMillis()
     * to ensure accurate validation even if user changes device time
     */
    private fun calculateRemainingDays(details: CampaignDetails): Int {
        val expireDate = details.expireDate ?: return Int.MAX_VALUE
        val currentDate = details.currentDate ?: System.currentTimeMillis() // Fallback only
        val remainingTime = expireDate - currentDate
        return (remainingTime / (24 * 60 * 60 * 1000)).toInt()
    }
}
```

---

## ReadyToUse Status Code Mapping Table

| Ready Status | Code | Message Pattern | Error Type | User Action | UI Behavior |
|--------------|------|-----------------|------------|-------------|-------------|
| `false` | `"1"` | "Campaign sold out" | `SOLD_OUT` | Find similar campaigns | Show sold out badge |
| `false` | `"2"` | "Max redemption per person reached" | `MAX_PER_PERSON` | Show redemption history | Display usage limit |
| `false` | `"3"` | "Campaign in cool down period" | `COOL_DOWN` | Show next available time | Display countdown |
| `false` | `"1403"` | "Condition invalid" | `CONDITION_INVALID` | Show condition details | Display requirements |
| `false` | `"1406"` | "Sponsor only campaign" | `SPONSOR_ONLY` | Upgrade membership | Show upgrade prompt |
| `false` | `"1409"` | "Campaign expired" | `EXPIRED` | Suggest alternatives | Show expiry date |
| `false` | `"1410"` | "Campaign not started yet" | `CAMPAIGN_PENDING` | Show start date | Display countdown |
| `false` | `"1416"` | "App version expired" | `VERSION_EXPIRED` | Redirect to app store | Show update dialog |
| `false` | `"1427"` | "Terms and conditions violation" | `TERMS_VIOLATION` | Review terms & conditions | Display terms violation |
| `false` | `null` | "Not Authenticated" | `DEVICE_LOGIN` | No action needed | Hide redeem button |
| `false` | `null` | "Campaign Expired" | `EXPIRED` | Suggest alternatives | Show expiry date |
| `false` | `null` | "Campaign sold out" | `SOLD_OUT` | Find similar campaigns | Show sold out badge |
| `false` | `null` | Contains "point" | `INSUFFICIENT_POINTS` | Show point purchase | Display point shortage |
| `false` | `null` | Custom message | `CUSTOM_CONDITION` | Follow server message | Show server message |
| `true` | `null` | `null` | `SUCCESS` | Enable redemption | Show redeem button |

---

## Enhanced Error Classes

```kotlin
sealed class CampaignValidationError(val message: String, val errorId: String) {
    // Priority 1: Login validation
    data class DeviceLogin(val msg: String) : CampaignValidationError(msg, "DEVICE_LOGIN")
    
    // Priority 2: Points validation
    data class InsufficientPoints(val msg: String) : CampaignValidationError(msg, "INSUFFICIENT_POINTS")
    
    // Priority 3: Campaign state validation (when isConditionPass = true)
    data class Expired(val msg: String) : CampaignValidationError(msg, "EXPIRED")
    data class SoldOut(val msg: String) : CampaignValidationError(msg, "SOLD_OUT")
    
    // Priority 4: Condition alerts (when isConditionPass = false)
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

## Modern Implementation Example

```kotlin
class CampaignDetailActivity : AppCompatActivity() {
    
    private val validator = ModernCampaignValidator()
    private val campaignUseCase = BuzzebeesSDK.instance().campaignUseCase
    private lateinit var redeemButton: Button
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_campaign_detail)
        
        redeemButton = findViewById(R.id.btn_redeem)
        
        // Step 1: Load campaign details
        val campaignId = intent.getStringExtra("campaign_id") ?: return
        loadCampaignDetail(campaignId)
    }
    
    private fun loadCampaignDetail(campaignId: String) {
        showLoading()
        
        // Step 1: Call getCampaignDetail() - SDK automatically calculates readyToUse
        campaignUseCase.getCampaignDetail(campaignId) { result ->
            hideLoading()
            
            when (result) {
                is CampaignResult.SuccessCampaignDetail -> {
                    val campaignDetails = result.result
                    displayCampaignInfo(campaignDetails)
                    
                    // Step 2: Use SDK's readyToUse validation
                    validateCampaignAndSetupUI(campaignDetails)
                }
                is CampaignResult.Error -> {
                    showErrorAndFinish("Failed to load campaign: ${result.error.message}")
                }
            }
        }
    }
    
    private fun validateCampaignAndSetupUI(campaignDetails: CampaignDetails) {
        val validationResult = validator.validateCampaignForRedemption(campaignDetails)
        
        when (validationResult) {
            is CampaignValidationResult.Success -> {
                // ✅ Ready to redeem - Enable redeem button
                setupRedeemButton(campaignDetails, enabled = true)
            }
            is CampaignValidationResult.Error -> {
                // ❌ Not ready - Handle based on readyToUse error type
                handleReadyToUseError(validationResult.error, campaignDetails)
            }
        }
    }
    
    private fun setupRedeemButton(campaignDetails: CampaignDetails, enabled: Boolean) {
        redeemButton.isEnabled = enabled
        
        if (enabled) {
            redeemButton.text = "Redeem (${campaignDetails.pointPerUnit} points)"
            redeemButton.setOnClickListener {
                // Step 3: Call redeem when button clicked
                performRedemption(campaignDetails.id.toString())
            }
        }
    }
    
    private fun handleReadyToUseError(error: CampaignValidationError, campaignDetails: CampaignDetails) {
        when (error) {
            is CampaignValidationError.DeviceLogin -> {
                // Hide redeem button for device users
                redeemButton.visibility = View.GONE
                showInfoMessage("Please login to redeem campaigns")
            }
            
            is CampaignValidationError.InsufficientPoints -> {
                // Disable button and show points needed
                redeemButton.isEnabled = false
                redeemButton.text = "Insufficient Points"
                showPointsNeededAlert(campaignDetails.pointPerUnit ?: 0.0)
            }
            
            is CampaignValidationError.SoldOut -> {
                // Disable button and show sold out status
                redeemButton.isEnabled = false
                redeemButton.text = "Sold Out"
                showSoldOutAlert()
            }
            
            is CampaignValidationError.Expired -> {
                // Disable button and show expiry info
                redeemButton.isEnabled = false
                redeemButton.text = "Expired"
                showExpiryAlert(campaignDetails.expireDate)
            }
            
            is CampaignValidationError.VersionExpired -> {
                // Show update dialog
                redeemButton.isEnabled = false
                showUpdateAppDialog()
            }
            
            is CampaignValidationError.TermsViolation -> {
                // Show terms violation dialog
                redeemButton.isEnabled = false
                showTermsViolationDialog()
            }
            
            is CampaignValidationError.CustomCondition -> {
                // Show server message from readyToUse.message
                redeemButton.isEnabled = false
                redeemButton.text = "Not Available"
                showCustomAlert(error.message)
            }
            
            else -> {
                // Handle other error types
                redeemButton.isEnabled = false
                redeemButton.text = "Cannot Redeem"
                showGenericAlert(error.message)
            }
        }
    }
    
    private fun performRedemption(campaignId: String) {
        showLoading()
        
        campaignUseCase.redeem(campaignId) { result ->
            hideLoading()
            
            when (result) {
                is CampaignResult.SuccessRedeem -> {
                    showRedemptionSuccess(result.result)
                }
                is CampaignResult.Error -> {
                    showRedemptionError(result.error)
                }
            }
        }
    }
```
    
    // UI Helper Methods
    private fun showPointsNeededAlert(pointsNeeded: Double) {
        AlertDialog.Builder(this)
            .setTitle("Insufficient Points")
            .setMessage("You need ${pointsNeeded.toInt()} points to redeem this campaign.")
            .setPositiveButton("Get Points") { _, _ -> 
                // Navigate to points purchase
            }
            .setNegativeButton("OK", null)
            .show()
    }
    
    private fun showUpdateAppDialog() {
        AlertDialog.Builder(this)
            .setTitle("Update Required")
            .setMessage("Please update your app to redeem this campaign.")
            .setPositiveButton("Update") { _, _ ->
                // Open app store
            }
            .setCancelable(false)
            .show()
    }
    
    private fun showTermsViolationDialog() {
        AlertDialog.Builder(this)
            .setTitle("Terms Violation")
            .setMessage("This privilege cannot be redeemed under the specified terms and conditions.")
            .setPositiveButton("Review Terms") { _, _ ->
                // Open terms and conditions
            }
            .setNegativeButton("OK", null)
            .show()
    }
}
```

### Key Implementation Points

```kotlin
// Modern SDK Flow Implementation
class CampaignFlow {
    
    suspend fun executeCampaignFlow(campaignId: String): CampaignFlowResult {
        // Step 1: Get campaign details (SDK calculates readyToUse automatically)
        val campaignDetails = getCampaignDetails(campaignId) 
            ?: return CampaignFlowResult.Error("Campaign not found")
        
        // Step 2: Use SDK's readyToUse validation
        val readyToUse = campaignDetails.readyToUse
        
        // Step 3: Handle result based on readyToUse status
        return if (readyToUse.isReadyToUse == true) {
            CampaignFlowResult.ReadyToRedeem(campaignDetails)
        } else {
            val error = mapReadyToUseToError(readyToUse.code, readyToUse.message)
            CampaignFlowResult.ValidationFailed(error)
        }
    }
    
    private fun mapReadyToUseToError(code: String?, message: String?): CampaignValidationError {
        // Use the ModernCampaignValidator mapping logic
        return ModernCampaignValidator().mapReadyToUseCodeToError(
            code = code,
            message = message ?: "Campaign not available"
        )
    }
}

sealed class CampaignFlowResult {
    data class ReadyToRedeem(val campaignDetails: CampaignDetails) : CampaignFlowResult()
    data class ValidationFailed(val error: CampaignValidationError) : CampaignFlowResult()
    data class Error(val message: String) : CampaignFlowResult()
}
```

---

## Summary

This Campaign Handling & Validation Guide provides **modern SDK-based validation logic** using the built-in `readyToUse` field that shows developers how to properly handle campaign details and implement validation that matches real Buzzebees application behavior.

### Key Features

- **🆕 SDK Auto-Validation**: Use the built-in `readyToUse` field calculated by SDK
- **🎯 Simplified Integration**: No need for complex manual validation logic
- **📱 Production-Ready**: Exact validation results used in live applications
- **🔧 Easy Implementation**: Clear guidance on how to handle each readyToUse status in the UI
- **⚡ Consistent Results**: Server-side validation ensures accuracy across all clients

### Migration Benefits

**Old Approach** (Manual Validation):
- Complex client-side validation logic
- Risk of inconsistency between apps
- Manual calculation of expiry, quantity, etc.
- Prone to client-time manipulation

**New Approach** (SDK ReadyToUse):
- ✅ **Server-calculated validation**
- ✅ **Consistent across all platforms**
- ✅ **Secure server-time based calculations**
- ✅ **Simple client-side implementation**
- ✅ **Automatic updates when business rules change**

### Integration Points

- **Extends**: [CampaignUseCase Guide](CampaignUseCase_GUIDE.md) with readyToUse validation
- **Uses**: Built-in `readyToUse` field from `getCampaignDetail()` method
- **Provides**: Complete validation framework using SDK's auto-calculated status

**Purpose**: This guide focuses on **how to use the SDK's readyToUse validation** - checking the right fields (`isReadyToUse`, `message`, `code`) and providing appropriate user feedback based on campaign readiness status.
