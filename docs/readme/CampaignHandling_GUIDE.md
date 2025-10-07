# Campaign Handling & Validation Guide

This guide covers campaign redemption validation flow and error handling patterns for the Buzzebees SDK.

---

## Overview

The campaign handling system implements a validation process following the standard flow: **call detail → handle readyToUse status → if ready → call redeem, else show appropriate UI**. This validation flow ensures data integrity and provides clear user feedback for all campaign states.

**SDK Enhancement**: The `getCampaignDetail()` method automatically calculates and includes a `readyToUse` status object that validates campaign conditions. This eliminates the need for complex client-side validation logic.

### Standard Campaign Flow

All implementations should follow this sequence:

1. **Call `getCampaignDetail()`** - Get campaign information from server
2. **Check `readyToUse` status** - SDK automatically validates business rules  
3. **Handle readyToUse result**:
   - ✅ **`isReadyToUse = true`** → Enable redeem button → Call `redeem()` when clicked
   - ❌ **`isReadyToUse = false`** → Show appropriate UI based on `message` and `code`

### SDK-Calculated ReadyToUse

The `readyToUse` field in `CampaignDetails` is automatically calculated by the SDK:

```kotlin
data class ReadyToUse(
    var isReadyToUse: Boolean? = false,  // Campaign is ready for redemption
    var message: String? = null,         // Status message (reason if not ready)
    var code: String? = null             // Status code identifier
)
```

**SDK Auto-Validation**: The SDK automatically checks:
- User authentication status
- Campaign expiration using server time
- Available quantity and sold count
- User eligibility and condition pass status
- All condition alerts and business rules

---

## ReadyToUse Validation Approach

**Recommended Approach**: Use the SDK's built-in `readyToUse` validation for simplicity and consistency.

```kotlin
class CampaignValidator {
    
    fun validateCampaignForRedemption(campaignDetails: CampaignDetails): CampaignValidationResult {
        val readyToUse = campaignDetails.readyToUse
        
        return if (readyToUse?.isReadyToUse == true) {
            CampaignValidationResult.Success
        } else {
            val error = mapReadyToUseCodeToError(
                code = readyToUse?.code,
                message = readyToUse?.message ?: "Campaign not available"
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
                when {
                    message.contains("authenticated", ignoreCase = true) ->
                        CampaignValidationError.DeviceLogin(message)
                    message.contains("expired", ignoreCase = true) ->
                        CampaignValidationError.Expired(message)
                    message.contains("sold out", ignoreCase = true) ->
                        CampaignValidationError.SoldOut(message)
                    else -> CampaignValidationError.CustomCondition(message)
                }
            }
        }
    }
}
```

## Optional: Enhanced Validation with Points

For better user experience, you can add points validation as an additional layer:

```kotlin
class EnhancedCampaignValidator(
    private val profileUseCase: IProfileUseCase
) {
    
    suspend fun validateCampaignWithPoints(campaignDetails: CampaignDetails): CampaignValidationResult {
        // Step 1: Check readyToUse status first
        val readyToUse = campaignDetails.readyToUse
        
        if (readyToUse?.isReadyToUse != true) {
            val error = mapReadyToUseCodeToError(
                code = readyToUse?.code,
                message = readyToUse?.message ?: "Campaign not available"
            )
            return CampaignValidationResult.Error(error)
        }
        
        // Step 2: Optional points validation
        if (shouldCheckPoints(campaignDetails)) {
            val pointsValidation = validatePoints(campaignDetails)
            if (!pointsValidation.isValid) {
                return CampaignValidationResult.Error(
                    CampaignValidationError.InsufficientPoints(pointsValidation.message)
                )
            }
        }
        
        return CampaignValidationResult.Success
    }
    
    private suspend fun validatePoints(campaignDetails: CampaignDetails): PointsValidationResult {
        return try {
            val requiredPoints = campaignDetails.pointPerUnit ?: 0.0
            if (requiredPoints <= 0) return PointsValidationResult(true)
            
            val currentPoints = getCurrentUserPoints()
            if (currentPoints < requiredPoints) {
                PointsValidationResult(
                    false, 
                    "You need ${requiredPoints.toInt()} points to redeem this campaign"
                )
            } else {
                PointsValidationResult(true)
            }
        } catch (e: Exception) {
            PointsValidationResult(true) // Don't block redemption on points error
        }
    }
    
    private fun shouldCheckPoints(campaignDetails: CampaignDetails): Boolean {
        val requiredPoints = campaignDetails.pointPerUnit ?: 0.0
        return requiredPoints > 0
    }
    
    private suspend fun getCurrentUserPoints(): Double {
        return try {
            val profile = profileUseCase.getInternalProfile()
            when (profile) {
                is ProfileResult.SuccessInternalProfile -> profile.result.currentPoints ?: 0.0
                else -> 0.0
            }
        } catch (e: Exception) {
            0.0
        }
    }
}

data class PointsValidationResult(
    val isValid: Boolean,
    val message: String = ""
)
```

---

## ReadyToUse Status Code Reference

| Ready Status | Code | Message Pattern | Error Type |
|--------------|------|-----------------|------------|
| `false` | `"1"` | "Campaign sold out" | `SOLD_OUT` |
| `false` | `"2"` | "Max redemption per person reached" | `MAX_PER_PERSON` |
| `false` | `"3"` | "Campaign in cool down period" | `COOL_DOWN` |
| `false` | `"1403"` | "Condition invalid" | `CONDITION_INVALID` |
| `false` | `"1406"` | "Sponsor only campaign" | `SPONSOR_ONLY` |
| `false` | `"1409"` | "Campaign expired" | `EXPIRED` |
| `false` | `"1410"` | "Campaign not started yet" | `CAMPAIGN_PENDING` |
| `false` | `"1416"` | "App version expired" | `VERSION_EXPIRED` |
| `false` | `"1427"` | "Terms and conditions violation" | `TERMS_VIOLATION` |
| `false` | `null` | "Not Authenticated" | `DEVICE_LOGIN` |
| `false` | `null` | Custom server message | `CUSTOM_CONDITION` |
| `true` | `null` | `null` | `SUCCESS` |

---

## Error Classes

```kotlin
sealed class CampaignValidationError(val message: String, val errorId: String) {
    data class DeviceLogin(val msg: String) : CampaignValidationError(msg, "DEVICE_LOGIN")
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
    data class InsufficientPoints(val msg: String) : CampaignValidationError(msg, "INSUFFICIENT_POINTS")
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
    private val campaignUseCase = BuzzebeesSDK.instance().campaignUseCase
    private lateinit var redeemButton: Button
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_campaign_detail)
        
        redeemButton = findViewById(R.id.btn_redeem)
        
        val campaignId = intent.getStringExtra("campaign_id") ?: return
        loadCampaignDetail(campaignId)
    }
    
    private fun loadCampaignDetail(campaignId: String) {
        campaignUseCase.getCampaignDetail(campaignId) { result ->
            when (result) {
                is CampaignResult.SuccessCampaignDetail -> {
                    val campaignDetails = result.result
                    displayCampaignInfo(campaignDetails)
                    handleValidation(campaignDetails)
                }
                is CampaignResult.Error -> {
                    showError("Failed to load campaign: ${result.error.message}")
                }
            }
        }
    }
    
    private fun handleValidation(campaignDetails: CampaignDetails) {
        val validationResult = validator.validateCampaignForRedemption(campaignDetails)
        
        when (validationResult) {
            is CampaignValidationResult.Success -> {
                setupRedeemButton(campaignDetails, enabled = true)
            }
            is CampaignValidationResult.Error -> {
                handleValidationError(validationResult.error, campaignDetails)
            }
        }
    }
    
    private fun setupRedeemButton(campaignDetails: CampaignDetails, enabled: Boolean) {
        redeemButton.isEnabled = enabled
        
        if (enabled) {
            val pointText = campaignDetails.pointPerUnit?.let { 
                if (it > 0) " (${it.toInt()} points)" else "" 
            } ?: ""
            redeemButton.text = "Redeem$pointText"
            redeemButton.setOnClickListener {
                performRedemption(campaignDetails.id.toString())
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
            }
            is CampaignValidationError.CampaignPending -> {
                redeemButton.text = "Not Started"
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
    
    private fun performRedemption(campaignId: String) {
        campaignUseCase.redeem(campaignId) { result ->
            when (result) {
                is CampaignResult.SuccessRedeem -> {
                    showRedemptionSuccess(result.result)
                }
                is CampaignResult.Error -> {
                    showError(result.error.message ?: "Redemption failed")
                }
            }
        }
    }
    
    private fun showPointsDialog(pointsNeeded: Double) {
        AlertDialog.Builder(this)
            .setTitle("Insufficient Points")
            .setMessage("You need ${pointsNeeded.toInt()} points to redeem this campaign.")
            .setPositiveButton("Get Points") { _, _ -> /* Navigate to points */ }
            .setNegativeButton("OK", null)
            .show()
    }
    
    private fun showUpdateDialog() {
        AlertDialog.Builder(this)
            .setTitle("Update Required")
            .setMessage("Please update your app to redeem this campaign.")
            .setPositiveButton("Update") { _, _ -> /* Open app store */ }
            .show()
    }
    
    private fun showSponsorDialog() {
        AlertDialog.Builder(this)
            .setTitle("Sponsor Members Only")
            .setMessage("This campaign is only available for sponsor members.")
            .setPositiveButton("Learn More") { _, _ -> /* Navigate to sponsor info */ }
            .setNegativeButton("OK", null)
            .show()
    }
}
```

### Campaign Flow Summary

```kotlin
class CampaignFlow {
    
    suspend fun executeCampaignFlow(campaignId: String): CampaignFlowResult {
        // Get campaign details with readyToUse calculated by SDK
        val campaignDetails = getCampaignDetails(campaignId) 
            ?: return CampaignFlowResult.Error("Campaign not found")
        
        // Handle readyToUse status
        val readyToUse = campaignDetails.readyToUse
        
        return if (readyToUse?.isReadyToUse == true) {
            CampaignFlowResult.ReadyToRedeem(campaignDetails)
        } else {
            val error = mapReadyToUseToError(readyToUse?.code, readyToUse?.message)
            CampaignFlowResult.ValidationFailed(error)
        }
    }
}
```

---

## Summary

This Campaign Handling & Validation Guide provides SDK-based validation using the built-in `readyToUse` field for proper campaign handling.

### Key Features

- **SDK Auto-Validation**: Use the built-in `readyToUse` field calculated by SDK
- **Simple Integration**: Clear guidance on handling each readyToUse status
- **Optional Enhancement**: Points validation available for improved UX
- **Production-Ready**: Validation results used in live applications
- **Server-Side Calculation**: Secure validation prevents client manipulation

### Integration Options

1. **Basic Integration**: Use `CampaignValidator` - handles all readyToUse cases
2. **Enhanced Integration**: Use `EnhancedCampaignValidator` - adds optional points validation

### Benefits

- ✅ **Server-calculated validation** ensures accuracy
- ✅ **Consistent across platforms**
- ✅ **Simple client implementation**
- ✅ **Flexible points validation**
- ✅ **Automatic business rule updates**

**Purpose**: This guide shows how to properly use the SDK's readyToUse validation for campaign handling with optional points validation enhancement.
