## RegisterUseCase Guide

This guide shows how to initialize and use every public method in `RegisterUseCase`, with suspend
and callback examples where available. The RegisterUseCase provides comprehensive user registration
functionality for creating new user accounts with complete profile data, device information,
marketing preferences, and privacy consent management.

### Getting an instance

```kotlin
val registerService = BuzzebeesSDK.instance().registerUseCase
```

---

### register

Registers a new user with comprehensive registration data including device information, marketing preferences, and privacy consent management.

- Request (caller-supplied)

| Field Name     | Description                                   | Mandatory | Data Type    |
|----------------|-----------------------------------------------|-----------|--------------|
| registerForm   | Complete registration form with user data    | M         | RegisterForm |

### RegisterForm Fields

| Field Name               | Description                        | Mandatory | Data Type         |
|--------------------------|------------------------------------|-----------|-------------------|
| deviceNotificationEnable | Device notification enabled flag   | M         | Boolean           |
| username                 | Username for login                  | M         | String            |
| password                 | User password                       | M         | String            |
| confirmPassword          | Password confirmation               | M         | String            |
| firstname                | First name                          | M         | String            |
| lastname                 | Last name                           | M         | String            |
| contactNumber            | Phone number                        | M         | String            |
| otp                      | OTP verification code               | M         | String            |
| refCode                  | OTP reference code                  | M         | String            |
| address                  | Street address                      | O         | String?           |
| gender                   | Gender                              | O         | String?           |
| birthdate                | Birth date timestamp                | O         | Long?             |
| email                    | Email address                       | O         | String?           |
| refUserCode              | Referral user code                  | O         | String?           |
| termAndConditionVersion  | Terms & conditions version accepted | O         | String?           |
| dataPrivacyVersion       | Data privacy version accepted       | O         | String?           |
| marketingOptionsVersion  | Marketing options version accepted  | O         | String?           |
| emailMarketing           | Email marketing consent (0/1)       | O         | Int?              |
| smsMarketing             | SMS marketing consent (0/1)         | O         | Int?              |
| notificationMarketing    | Push notification consent (0/1)     | O         | Int?              |
| lineMarketing            | LINE marketing consent (0/1)        | O         | Int?              |
| phoneMarketing           | Phone marketing consent (0/1)       | O         | Int?              |
| options                  | Additional custom options           | O         | Map<String,String>|

- Response (`RegisterResponse`)
  HTTP status: 200

### RegisterResponse Entity Fields

| Field Name     | Description               | Data Type | JSON Field     |
|----------------|---------------------------|-----------|----------------|
| status         | Registration status       | String?   | status         |
| username       | Registered username       | String?   | username       |
| userId         | User unique identifier    | String?   | userId         |
| firstname      | First name                | String?   | firstname      |
| lastname       | Last name                 | String?   | lastname       |
| address        | Address                   | String?   | address        |
| email          | Email address             | String?   | email          |
| clientVersion  | Client version            | String?   | client_version |
| carrier        | Mobile carrier            | String?   | carrier        |
| macAddress     | Device MAC address        | String?   | mac_address    |
| platform       | Device platform           | String?   | platform       |
| os             | Operating system          | String?   | os             |
| gender         | Gender                    | String?   | gender         |
| birthdate      | Birth date timestamp      | Long?     | birthdate      |

- Usage

```kotlin
// Create registration form
val registerForm = RegisterForm(
    deviceNotificationEnable = true,
    username = "john_doe_2024",
    password = "SecurePassword123!",
    confirmPassword = "SecurePassword123!",
    firstname = "John",
    lastname = "Doe",
    contactNumber = "0812345678",
    otp = "123456",
    refCode = "REF123456",
    address = "123 Main Street, Bangkok",
    gender = "Male",
    birthdate = System.currentTimeMillis() - (25 * 365 * 24 * 60 * 60 * 1000L), // 25 years ago
    email = "john.doe@example.com",
    refUserCode = "FRIEND_REF_001",
    termAndConditionVersion = "v2.1",
    dataPrivacyVersion = "v1.3",
    marketingOptionsVersion = "v1.0",
    emailMarketing = 1, // Accept email marketing
    smsMarketing = 1,   // Accept SMS marketing
    notificationMarketing = 1, // Accept push notifications
    lineMarketing = 0,  // Decline LINE marketing
    phoneMarketing = 0, // Decline phone marketing
    options = mapOf(
        "utm_source" to "google_ads",
        "utm_campaign" to "new_user_2024"
    )
)

// Suspend
val result = registerService.register(registerForm)

// Callback
registerService.register(registerForm) { result ->
    when (result) {
        is RegisterResult.Success -> {
            // Handle successful registration
            val registerResponse = result.result
            println("Registration successful!")
            println("User ID: ${registerResponse.userId}")
            println("Username: ${registerResponse.username}")
            println("Welcome ${registerResponse.firstname} ${registerResponse.lastname}!")
            
            // Registration completed - user can now login
        }
        is RegisterResult.Error -> {
            // Handle registration error
            val errorCode = result.error.code
            val errorMessage = result.error.message
            println("Registration failed: $errorMessage")
        }
    }
}
```

#### Marketing Consent Values

Marketing consent fields accept integer values:

| Value | Description |
|-------|-------------|
| 1     | Accept      |
| 0     | Decline     |
| null  | Not specified |

#### Error Handling

Common registration errors and their handling:

| Error Code | Error ID | Scenario                     | User Message                              | Recommended Action                    |
|------------|----------|------------------------------|-------------------------------------------|---------------------------------------|
| 409        | 1904     | Username already exists      | "This username is already taken"          | Let user choose different username    |
| 409        | 2078     | Phone number already exists  | "This phone number is already registered"| Let user verify ownership or use different number |
| 409        | 2091     | Invalid OTP                  | "Invalid OTP code"                        | Let user request new OTP              |
| 400        | -        | Password validation failed   | "Password must be at least 8 characters"  | Show password requirements            |
| 400        | -        | Password mismatch            | "Passwords do not match"                  | Let user re-enter confirmation       |
| 400        | -        | Invalid phone format         | "Invalid phone number format"             | Show correct format example           |
| 400        | -        | Invalid email format         | "Invalid email address"                   | Let user correct email format         |
| 429        | -        | Too many attempts            | "Too many registration attempts"          | Show cooldown timer                   |

#### Implementation Example

```kotlin
// Comprehensive error handling example
registerService.register(registerForm) { result ->
    when (result) {
        is RegisterResult.Success -> {
            // Registration successful
            val response = result.result
            showRegistrationSuccess(response)
            navigateToLoginScreen()
        }
        is RegisterResult.Error -> {
            val action = when {
                result.error.code == "409" && result.error.id == "1904" ->
                    ErrorAction.ShowUsernameExistsError()
                result.error.code == "409" && result.error.id == "2078" ->
                    ErrorAction.ShowPhoneExistsError()
                result.error.code == "409" && result.error.id == "2091" ->
                    ErrorAction.ShowInvalidOtpError()
                result.error.code == "400" && result.error.message?.contains("password", true) == true ->
                    ErrorAction.ShowPasswordValidationError()
                result.error.code == "400" && result.error.message?.contains("email", true) == true ->
                    ErrorAction.ShowEmailValidationError()
                result.error.code == "429" ->
                    ErrorAction.ShowRateLimitError()
                else ->
                    ErrorAction.ShowGenericError(result.error.message)
            }
            handleErrorAction(action)
        }
    }
}
```

#### Registration Flow Best Practices

1. **Phone Verification**: Always verify phone numbers with OTP before registration
2. **Password Validation**: Implement client-side password strength validation
3. **Marketing Consent**: Clearly present marketing options with opt-in checkboxes
4. **Privacy Compliance**: Include links to privacy policy and terms of service
5. **Error Recovery**: Provide clear error messages and recovery options
6. **Success Handling**: Guide users to login screen after successful registration

#### Complete Registration Example

```kotlin
class RegistrationViewModel {
    
    fun registerUser(
        username: String,
        password: String,
        confirmPassword: String,
        firstName: String,
        lastName: String,
        phoneNumber: String,
        otp: String,
        refCode: String,
        email: String?,
        acceptsEmailMarketing: Boolean,
        acceptsSmsMarketing: Boolean,
        acceptsPushNotifications: Boolean
    ) {
        // Validate inputs
        if (password != confirmPassword) {
            showError("Passwords do not match")
            return
        }
        
        // Create registration form
        val registerForm = RegisterForm(
            deviceNotificationEnable = acceptsPushNotifications,
            username = username,
            password = password,
            confirmPassword = confirmPassword,
            firstname = firstName,
            lastname = lastName,
            contactNumber = phoneNumber,
            otp = otp,
            refCode = refCode,
            email = email,
            emailMarketing = if (acceptsEmailMarketing) 1 else 0,
            smsMarketing = if (acceptsSmsMarketing) 1 else 0,
            notificationMarketing = if (acceptsPushNotifications) 1 else 0,
            termAndConditionVersion = "v2.1",
            dataPrivacyVersion = "v1.3",
            marketingOptionsVersion = "v1.0"
        )
        
        // Show loading
        showLoading()
        
        // Register user
        BuzzebeesSDK.instance().registerUseCase.register(registerForm) { result ->
            hideLoading()
            when (result) {
                is RegisterResult.Success -> {
                    showRegistrationSuccess()
                    navigateToLogin()
                }
                is RegisterResult.Error -> {
                    handleRegistrationError(result.error)
                }
            }
        }
    }
}
```

---

## Summary

The RegisterUseCase provides comprehensive user registration functionality within the Buzzebees SDK. It handles complete user onboarding including personal information collection, device information capture, marketing consent management, and privacy compliance. The UseCase supports OTP verification for phone number validation and includes extensive error handling for common registration scenarios.

The RegisterUseCase includes **1 method** with **2 variants**:
- `register(RegisterForm)` - Complete user registration with comprehensive data collection
- `register(RegisterForm, callback)` - Callback version for non-coroutine usage

**Key Features:**
- **Comprehensive Data Collection**: Personal info, contact details, preferences
- **Device Information Capture**: Platform, OS, carrier, and device identifiers
- **Marketing Consent Management**: Granular control over marketing channels
- **Privacy Compliance**: Version tracking for terms, privacy policy, and marketing options
- **OTP Verification**: Phone number validation with reference code system
- **Flexible Options**: Custom parameters through options map
- **Robust Error Handling**: Detailed error responses for various failure scenarios

The registration process ensures compliance with privacy regulations while providing a smooth onboarding experience for new users.
