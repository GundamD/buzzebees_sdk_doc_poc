# Error Handler Implementation Guide

This guide provides a complete implementation of error handling system for Buzzebees SDK to help developers properly handle errors and provide good user experience.

## ErrorAction Sealed Class

Create a sealed class to represent different error handling actions:

```kotlin
sealed class ErrorAction {
    // Generic error actions
    data class ShowError(val messageRes: Int) : ErrorAction()
    data class ShowGenericError(val message: String?) : ErrorAction()
    
    // Auth-related errors
    object ShowSupportContact : ErrorAction()
    object ShowAccountLockedDialog : ErrorAction()
    data class ShowSupportPhone(val phoneNumber: String) : ErrorAction()
    object ForceLogout : ErrorAction()
    
    // OTP-related errors
    object ShowMobileDuplicateError : ErrorAction()
    object ShowMobileFormatError : ErrorAction()
    data class ShowRateLimitError(val waitSeconds: Int) : ErrorAction()
    object ShowMobileNotFoundError : ErrorAction()
    object ShowInvalidOtpError : ErrorAction()
    object ResendOtpAutomatically : ErrorAction()
    data class LockOtpInput(val lockSeconds: Int) : ErrorAction()
    
    // Campaign-related errors
    object ShowSoldOutMessage : ErrorAction()
    object ShowMaxRedemptionReached : ErrorAction()
    object ShowAlreadyRedeemed : ErrorAction()
    object ShowCampaignUnavailable : ErrorAction()
    object ShowCampaignExpired : ErrorAction()
    object ShowCampaignNotStarted : ErrorAction()
    object ShowTermsAndConditionsDialog : ErrorAction()
    object ShowNotAuthorized : ErrorAction()
    object ShowPaymentError : ErrorAction()
    object ShowNoPermissionMessage : ErrorAction()
    object ShowEmptyState : ErrorAction()
    object ShowRetryOption : ErrorAction()
    
    // Profile-related errors
    object ShowTextTooLongError : ErrorAction()
    object ShowDuplicateEmailError : ErrorAction()
    object ShowDuplicateNationalIdError : ErrorAction()
    object ShowInvalidImageFormatError : ErrorAction()
    object ShowImageTooLargeError : ErrorAction()
    object ShowIncorrectCurrentPasswordError : ErrorAction()
    object ShowWeakPasswordError : ErrorAction()
    object ShowPasswordRecentlyUsedError : ErrorAction()
    object ShowInvalidMobileFormatError : ErrorAction()
    
    // Badge-related errors
    object ShowBadgeNotFound : ErrorAction()
    object ShowBadgeNoPermission : ErrorAction()
    object ShowBadgeConditionsNotMet : ErrorAction()
}
```

## Main Error Handler Class

Create a comprehensive error handler class:

```kotlin
class SDKErrorHandler(private val context: Context) {
    
    fun handleErrorAction(action: ErrorAction) {
        when (action) {
            // Generic errors
            is ErrorAction.ShowError -> {
                showDialog(context.getString(action.messageRes))
            }
            is ErrorAction.ShowGenericError -> {
                showDialog(action.message ?: context.getString(R.string.generic_error))
            }
            
            // Auth errors
            is ErrorAction.ShowSupportContact -> {
                showSupportContactDialog()
            }
            is ErrorAction.ShowAccountLockedDialog -> {
                showAccountLockedDialog()
            }
            is ErrorAction.ShowSupportPhone -> {
                showSupportPhoneDialog(action.phoneNumber)
            }
            is ErrorAction.ForceLogout -> {
                performLogout()
            }
            
            // OTP errors
            is ErrorAction.ShowMobileDuplicateError -> {
                showDialog("This number is already in use. Please use another number")
            }
            is ErrorAction.ShowMobileFormatError -> {
                showMobileFormatDialog()
            }
            is ErrorAction.ShowRateLimitError -> {
                showRateLimitDialog(action.waitSeconds)
            }
            is ErrorAction.ShowMobileNotFoundError -> {
                showMobileNotFoundDialog()
            }
            is ErrorAction.ShowInvalidOtpError -> {
                showInvalidOtpDialog()
            }
            is ErrorAction.ResendOtpAutomatically -> {
                resendOtpAutomatically()
            }
            is ErrorAction.LockOtpInput -> {
                lockOtpInput(action.lockSeconds)
            }
            
            // Campaign errors
            is ErrorAction.ShowSoldOutMessage -> {
                showCampaignSoldOutDialog()
            }
            is ErrorAction.ShowMaxRedemptionReached -> {
                showMaxRedemptionDialog()
            }
            is ErrorAction.ShowAlreadyRedeemed -> {
                showAlreadyRedeemedDialog()
            }
            is ErrorAction.ShowCampaignUnavailable -> {
                showDialog("This campaign is not available")
            }
            is ErrorAction.ShowCampaignExpired -> {
                showCampaignExpiredDialog()
            }
            is ErrorAction.ShowCampaignNotStarted -> {
                showCampaignNotStartedDialog()
            }
            is ErrorAction.ShowTermsAndConditionsDialog -> {
                showTermsAndConditionsDialog()
            }
            is ErrorAction.ShowNotAuthorized -> {
                showNotAuthorizedDialog()
            }
            is ErrorAction.ShowPaymentError -> {
                showPaymentErrorDialog()
            }
            is ErrorAction.ShowNoPermissionMessage -> {
                showDialog("No permission to view this campaign")
            }
            is ErrorAction.ShowEmptyState -> {
                showEmptyState()
            }
            is ErrorAction.ShowRetryOption -> {
                showRetryDialog()
            }
            
            // Profile errors
            is ErrorAction.ShowTextTooLongError -> {
                showDialog("Text is too long (maximum 100 characters)")
            }
            is ErrorAction.ShowDuplicateEmailError -> {
                showDialog("This email is already in use")
            }
            is ErrorAction.ShowDuplicateNationalIdError -> {
                showDialog("This national ID is already in use")
            }
            is ErrorAction.ShowInvalidImageFormatError -> {
                showDialog("Image must be JPG or PNG only")
            }
            is ErrorAction.ShowImageTooLargeError -> {
                showDialog("Image is too large (maximum 5MB)")
            }
            is ErrorAction.ShowIncorrectCurrentPasswordError -> {
                showDialog("Current password is incorrect")
            }
            is ErrorAction.ShowWeakPasswordError -> {
                showWeakPasswordDialog()
            }
            is ErrorAction.ShowPasswordRecentlyUsedError -> {
                showDialog("Cannot use recent password")
            }
            is ErrorAction.ShowInvalidMobileFormatError -> {
                showMobileFormatDialog()
            }
            
            // Badge errors
            is ErrorAction.ShowBadgeNotFound -> {
                showEmptyState()
            }
            is ErrorAction.ShowBadgeNoPermission -> {
                showDialog("No permission to view this badge")
            }
            is ErrorAction.ShowBadgeConditionsNotMet -> {
                showBadgeConditionsDialog()
            }
        }
    }
    
    // Helper methods for common dialogs
    private fun showDialog(message: String, title: String = "Notice") {
        AlertDialog.Builder(context)
            .setTitle(title)
            .setMessage(message)
            .setPositiveButton("OK", null)
            .show()
    }
    
    private fun showSupportContactDialog() {
        AlertDialog.Builder(context)
            .setTitle("Account Suspended")
            .setMessage("Your account has been suspended. Please contact customer service")
            .setPositiveButton("Contact Us") { _, _ ->
                openSupportContact()
            }
            .setNegativeButton("Cancel", null)
            .show()
    }
    
    private fun showAccountLockedDialog() {
        AlertDialog.Builder(context)
            .setTitle("Account Locked")
            .setMessage("Your account has been locked. Please reset your password or contact customer service")
            .setPositiveButton("Reset Password") { _, _ ->
                navigateToForgotPassword()
            }
            .setNegativeButton("Contact Us") { _, _ ->
                openSupportContact()
            }
            .show()
    }
    
    private fun showSupportPhoneDialog(phoneNumber: String) {
        AlertDialog.Builder(context)
            .setTitle("Account Suspended")
            .setMessage("Your account has been suspended. Please contact $phoneNumber")
            .setPositiveButton("Call") { _, _ ->
                makePhoneCall(phoneNumber)
            }
            .setNegativeButton("Cancel", null)
            .show()
    }
    
    private fun showMobileFormatDialog() {
        AlertDialog.Builder(context)
            .setTitle("Invalid Number Format")
            .setMessage("Please enter phone number in format 08XXXXXXXX")
            .setPositiveButton("OK", null)
            .show()
    }
    
    private fun showRateLimitDialog(waitSeconds: Int) {
        val message = "OTP requested too frequently. Please wait $waitSeconds seconds"
        showDialog(message)
        
        // Show countdown timer
        startCountdown(waitSeconds)
    }
    
    private fun showMobileNotFoundDialog() {
        AlertDialog.Builder(context)
            .setTitle("Number Not Found")
            .setMessage("Phone number not found in system. Please register first")
            .setPositiveButton("Register") { _, _ ->
                navigateToRegister()
            }
            .setNegativeButton("Cancel", null)
            .show()
    }
    
    private fun showInvalidOtpDialog() {
        AlertDialog.Builder(context)
            .setTitle("Invalid OTP")
            .setMessage("Please enter correct OTP code")
            .setPositiveButton("OK", null)
            .setNeutralButton("Request New Code") { _, _ ->
                requestNewOtp()
            }
            .show()
    }
    
    private fun showCampaignSoldOutDialog() {
        AlertDialog.Builder(context)
            .setTitle("Sold Out")
            .setMessage("This campaign is sold out")
            .setPositiveButton("View Other Campaigns") { _, _ ->
                showSimilarCampaigns()
            }
            .setNegativeButton("Close", null)
            .show()
    }
    
    private fun showMaxRedemptionDialog() {
        AlertDialog.Builder(context)
            .setTitle("Maximum Reached")
            .setMessage("You have reached the maximum redemption limit for this campaign")
            .setPositiveButton("View History") { _, _ ->
                showRedemptionHistory()
            }
            .setNeutralButton("View Other Campaigns") { _, _ ->
                showOtherCampaigns()
            }
            .setNegativeButton("Close", null)
            .show()
    }
    
    private fun showAlreadyRedeemedDialog() {
        AlertDialog.Builder(context)
            .setTitle("Already Redeemed")
            .setMessage("You have already redeemed this campaign")
            .setPositiveButton("View Details") { _, _ ->
                showRedemptionDetails()
            }
            .setNegativeButton("Close", null)
            .show()
    }
    
    private fun showCampaignExpiredDialog() {
        AlertDialog.Builder(context)
            .setTitle("Expired")
            .setMessage("This campaign has expired")
            .setPositiveButton("View New Campaigns") { _, _ ->
                showNewCampaigns()
            }
            .setNegativeButton("Close", null)
            .show()
    }
    
    private fun showCampaignNotStartedDialog() {
        AlertDialog.Builder(context)
            .setTitle("Not Started")
            .setMessage("This campaign has not started yet")
            .setPositiveButton("Set Reminder") { _, _ ->
                setNotificationReminder()
            }
            .setNegativeButton("Close", null)
            .show()
    }
    
    private fun showTermsAndConditionsDialog() {
        AlertDialog.Builder(context)
            .setTitle("Terms and Conditions")
            .setMessage("Please read and accept the terms and conditions")
            .setPositiveButton("Read Terms") { _, _ ->
                showTermsAndConditions()
            }
            .setNegativeButton("Cancel", null)
            .show()
    }
    
    private fun showNotAuthorizedDialog() {
        AlertDialog.Builder(context)
            .setTitle("Not Authorized")
            .setMessage("You are not authorized to redeem this campaign (VIP membership required)")
            .setPositiveButton("Upgrade") { _, _ ->
                showMembershipUpgrade()
            }
            .setNegativeButton("Close", null)
            .show()
    }
    
    private fun showPaymentErrorDialog() {
        AlertDialog.Builder(context)
            .setTitle("Payment Failed")
            .setMessage("Payment failed. Please check your payment information")
            .setPositiveButton("Retry") { _, _ ->
                retryPayment()
            }
            .setNeutralButton("Change Payment Method") { _, _ ->
                changePaymentMethod()
            }
            .setNegativeButton("Cancel", null)
            .show()
    }
    
    private fun showWeakPasswordDialog() {
        AlertDialog.Builder(context)
            .setTitle("Weak Password")
            .setMessage("Password must be at least 8 characters and contain numbers and letters")
            .setPositiveButton("OK", null)
            .show()
    }
    
    private fun showBadgeConditionsDialog() {
        AlertDialog.Builder(context)
            .setTitle("Conditions Not Met")
            .setMessage("You have not met the conditions for this badge yet")
            .setPositiveButton("View Conditions") { _, _ ->
                showBadgeConditions()
            }
            .setNegativeButton("Close", null)
            .show()
    }
    
    private fun showEmptyState() {
        // Show empty state view
        // Implementation depends on your UI framework
    }
    
    private fun showRetryDialog() {
        AlertDialog.Builder(context)
            .setTitle("Error")
            .setMessage("Something went wrong. Please try again")
            .setPositiveButton("Retry") { _, _ ->
                retryLastOperation()
            }
            .setNegativeButton("Cancel", null)
            .show()
    }
    
    // Action methods - implement based on your app's navigation and logic
    private fun performLogout() {
        // Implement logout logic
    }
    
    private fun navigateToForgotPassword() {
        // Navigate to forgot password screen
    }
    
    private fun openSupportContact() {
        // Open support contact (email, chat, etc.)
    }
    
    private fun makePhoneCall(phoneNumber: String) {
        val intent = Intent(Intent.ACTION_DIAL, Uri.parse("tel:$phoneNumber"))
        context.startActivity(intent)
    }
    
    private fun startCountdown(seconds: Int) {
        // Implement countdown timer
    }
    
    private fun navigateToRegister() {
        // Navigate to registration screen
    }
    
    private fun requestNewOtp() {
        // Request new OTP
    }
    
    private fun resendOtpAutomatically() {
        // Automatically resend OTP
    }
    
    private fun lockOtpInput(seconds: Int) {
        // Lock OTP input for specified seconds
    }
    
    private fun showSimilarCampaigns() {
        // Show similar campaigns
    }
    
    private fun showRedemptionHistory() {
        // Show redemption history
    }
    
    private fun showOtherCampaigns() {
        // Show other campaigns
    }
    
    private fun showRedemptionDetails() {
        // Show redemption details
    }
    
    private fun showNewCampaigns() {
        // Show new campaigns
    }
    
    private fun setNotificationReminder() {
        // Set notification reminder
    }
    
    private fun showTermsAndConditions() {
        // Show terms and conditions
    }
    
    private fun showMembershipUpgrade() {
        // Show membership upgrade options
    }
    
    private fun retryPayment() {
        // Retry payment
    }
    
    private fun changePaymentMethod() {
        // Change payment method
    }
    
    private fun showBadgeConditions() {
        // Show badge conditions
    }
    
    private fun retryLastOperation() {
        // Retry the last failed operation
    }
}
```

## Usage Example

Here's how to use the error handler in your application:

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var errorHandler: SDKErrorHandler
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        errorHandler = SDKErrorHandler(this)
    }
    
    private fun loginUser(username: String, password: String) {
        val auth = BuzzebeesSDK.instance().auth
        
        auth.loginUsernamePassword(username, password) { result ->
            when (result) {
                is AuthResult.SuccessLogin -> {
                    // Handle successful login
                    handleSuccessfulLogin(result.result)
                }
                is AuthResult.Error -> {
                    // Handle error using error handler
                    val action = when {
                        result.error.code == "401" && result.error.id == "401" ->
                            ErrorAction.ShowError(R.string.login_invalid_credentials)
                        result.error.code == "403" && result.error.id == "403" ->
                            ErrorAction.ShowSupportContact()
                        result.error.code == "409" && result.error.id == "1417" ->
                            ErrorAction.ShowAccountLockedDialog()
                        result.error.code == "409" && result.error.id == "1905" ->
                            ErrorAction.ShowSupportPhone("02-105-1000")
                        result.error.code == "409" && result.error.id == "2076" ->
                            ErrorAction.ForceLogout()
                        else ->
                            ErrorAction.ShowGenericError(result.error.message)
                    }
                    errorHandler.handleErrorAction(action)
                }
            }
        }
    }
}
```

## String Resources

Add these string resources to your `strings.xml`:

```xml
<resources>
    <string name="generic_error">Something went wrong. Please try again</string>
    <string name="login_invalid_credentials">Invalid password. Please try again</string>
    <string name="network_error">Network error. Please check your connection</string>
    <string name="server_error">Server not responding. Please try again later</string>
</resources>
```

## Benefits

1. **Consistent Error Handling**: Unified approach to handle errors across the entire app
2. **Better User Experience**: Meaningful error messages and appropriate actions
3. **Maintainable Code**: Centralized error handling logic
4. **Localizable**: Easy to support multiple languages
5. **Extensible**: Easy to add new error types and actions
6. **Testable**: Clear separation of concerns for better testing

## Best Practices

1. **Always provide meaningful error messages** for better user understanding
2. **Offer constructive actions** when possible (retry, contact support, etc.)
3. **Log errors for debugging** while showing user-friendly messages
4. **Test error scenarios** to ensure proper handling
5. **Keep error messages consistent** across the application
6. **Consider offline scenarios** and provide appropriate messaging

This error handling system provides a robust foundation for managing SDK errors while maintaining excellent user experience.