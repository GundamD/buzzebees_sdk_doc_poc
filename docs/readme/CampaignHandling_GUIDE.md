# Campaign Handling & Validation Guide

This comprehensive guide covers campaign redemption validation flow and error handling patterns for the Buzzebees SDK. It extends the **CampaignUseCase** with production-grade validation logic based on real application business rules.

---

## Overview

The campaign handling system implements a robust **validation process** following the standard flow: **call detail → validate → if no issue → call redeem, else show alert or disable button**. This validation flow ensures data integrity, prevents fraudulent activities, and provides clear user feedback for all campaign states and error conditions.

**Key Purpose**: Show developers how to properly implement the campaign detail validation flow that matches production Buzzebees application behavior.

### Standard Campaign Flow

The recommended implementation follows this sequence:

1. **Call `getCampaignDetail()`** - Get campaign information from server
2. **Validate campaign data** - Check all business rules and user eligibility  
3. **Handle validation result**:
   - ✅ **No issues** → Enable redeem button → Call `redeem()` when clicked
   - ❌ **Has issues** → Show appropriate alert or disable redeem button

### Required Understanding

To effectively implement this flow, developers need to understand these related components:

- **[CampaignUseCase Guide](CampaignUseCase_GUIDE.md)** - Core campaign methods (`getCampaignDetail`, `redeem`)
- **[ProfileUseCase Guide](ProfileUseCase_GUIDE.md)** - User profile data (`loginType`, `currentPoints`, user authentication status)
- **Campaign Business Logic** - Understanding of `isConditionPass`, `conditionAlertId`, and validation priority

**Why ProfileUseCase Matters**: Campaign validation requires user profile information to determine login type (Device vs Authenticated), current point balance, and user eligibility. The validation logic heavily depends on these profile attributes.

This guide focuses specifically on **validation logic** and **error handling patterns** that bridge campaign and profile data.

---

## Production Validation Decision Tree

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

## Validation Error Mapping Table

| Priority | Condition | Field Check | Error ID | User Action | UI Behavior |
|----------|-----------|-------------|-----------|-------------|-------------|
| 1 | Device Login | `loginType == LoginType.DEVICE` | `DEVICE_LOGIN` | No action needed | Hide redeem button |
| 2 | Insufficient Points | `currentPoints < pointPerUnit` | `INSUFFICIENT_POINTS` | Show point purchase | Display point shortage |
| 3a | Campaign Expired | `isConditionPass && remainingDays < 1` | `EXPIRED` | Suggest alternatives | Show expiry date |
| 3b | Quantity Zero | `isConditionPass && qty <= 0` | `SOLD_OUT` | Find similar campaigns | Show sold out badge |
| 3c | Item Count Sold Out | `isConditionPass && itemCountSold >= quantity` | `SOLD_OUT` | Find similar campaigns | Show sold out badge |
| 4.1 | Condition Alert 1 | `!isConditionPass && conditionAlertId == "1"` | `SOLD_OUT` | Find similar campaigns | Show sold out badge |
| 4.2 | Condition Alert 2 | `!isConditionPass && conditionAlertId == "2"` | `MAX_PER_PERSON` | Show redemption history | Display usage limit |
| 4.3 | Condition Alert 3 | `!isConditionPass && conditionAlertId == "3"` | `COOL_DOWN` | Show next available time | Display countdown |
| 4.4 | Condition Invalid | `!isConditionPass && conditionAlertId == "1403"` | `CONDITION_INVALID` | Show condition details | Display requirements |
| 4.5 | Sponsor Only | `!isConditionPass && conditionAlertId == "1406"` | `SPONSOR_ONLY` | Upgrade membership | Show upgrade prompt |
| 4.6 | Expired (Alert) | `!isConditionPass && conditionAlertId == "1409"` | `EXPIRED` | Suggest alternatives | Show expiry date |
| 4.7 | Campaign Pending | `!isConditionPass && conditionAlertId == "1410"` | `CAMPAIGN_PENDING` | Show start date | Display countdown |
| 4.8 | Version Expired | `!isConditionPass && conditionAlertId == "1416"` | `VERSION_EXPIRED` | Redirect to app store | Show update dialog |
| 4.9 | Terms Violation | `!isConditionPass && conditionAlertId == "1427"` | `TERMS_VIOLATION` | Review terms & conditions | Display terms violation |
| 4.10 | Custom Condition | `!isConditionPass && other conditionAlertId` | `CUSTOM_CONDITION` | Follow server message | Show server message |

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

## Production Implementation Example

```kotlin
class CampaignDetailActivity : AppCompatActivity() {
    
    private val validator = ProductionCampaignValidator()
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
        
        // Step 1: Call getCampaignDetail()
        campaignUseCase.getCampaignDetail(campaignId) { result ->
            hideLoading()
            
            when (result) {
                is CampaignResult.SuccessCampaignDetail -> {
                    val campaignDetails = result.result
                    displayCampaignInfo(campaignDetails)
                    
                    // Step 2: Validate campaign data
                    validateCampaignAndSetupUI(campaignDetails)
                }
                is CampaignResult.Error -> {
                    showErrorAndFinish("Failed to load campaign: ${result.error.message}")
                }
            }
        }
    }
    
    private fun validateCampaignAndSetupUI(campaignDetails: CampaignDetails) {
        val validationResult = validator.validateCampaignForRedemption(
            campaignDetails = campaignDetails,
            internalProfile = getCurrentUserProfile(),
            currentPoints = getUserCurrentPoints()
        )
        
        when (validationResult) {
            is CampaignValidationResult.Success -> {
                // ✅ No issues - Enable redeem button
                setupRedeemButton(campaignDetails, enabled = true)
            }
            is CampaignValidationResult.Error -> {
                // ❌ Has issues - Handle based on error type
                handleValidationError(validationResult.error, campaignDetails)
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
    
    private fun handleValidationError(error: CampaignValidationError, campaignDetails: CampaignDetails) {
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
                // Show server message
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
// Standard Flow Implementation
class CampaignFlow {
    
    suspend fun executeCampaignFlow(campaignId: String): CampaignFlowResult {
        // Step 1: Get campaign details
        val campaignDetails = getCampaignDetails(campaignId) 
            ?: return CampaignFlowResult.Error("Campaign not found")
        
        // Step 2: Validate
        val validationResult = validator.validateCampaignForRedemption(
            campaignDetails, userProfile, userPoints
        )
        
        // Step 3: Handle result
        return when (validationResult) {
            is CampaignValidationResult.Success -> {
                CampaignFlowResult.ReadyToRedeem(campaignDetails)
            }
            is CampaignValidationResult.Error -> {
                CampaignFlowResult.ValidationFailed(validationResult.error)
            }
        }
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

This Campaign Handling & Validation Guide provides **production-ready validation logic** that shows developers how to properly handle campaign details and implement validation that matches real Buzzebees application behavior.

### Key Features

- **🔒 Production Logic**: Exact validation flow used in live applications
- **🎯 Priority-Based Validation**: 12-step validation process with proper priority ordering  
- **📱 Real Error Handling**: Handle campaign details based on actual field values
- **🔧 Easy Integration**: Seamlessly integrates with existing CampaignUseCase methods
- **⚡ UI-Ready**: Clear guidance on how to handle each error type in the UI

### Integration Points

- **Extends**: [CampaignUseCase Guide](CampaignUseCase_GUIDE.md) with validation logic
- **Uses**: Existing `getCampaignDetail()` and `redeem()` methods from CampaignUseCase
- **Provides**: Complete validation framework for campaign redemption flows

**Purpose**: This guide focuses on **how to handle campaign details properly** - checking the right fields, applying the correct business logic, and providing appropriate user feedback based on campaign state and user conditions.
