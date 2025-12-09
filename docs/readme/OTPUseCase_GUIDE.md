## OTPUseCase Guide

This guide shows how to initialize and use every public method in `OTPUseCase`, with suspend and
callback examples where available.

### Getting an instance

```kotlin
val otp = BuzzebeesSDK.instance().otp
```

---

## OTP Channel Types

The `channel` parameter in OTP methods specifies the purpose/context of the OTP request. Use these predefined values:

| Channel Value       | Description                      | Use Case                     |
|---------------------|----------------------------------|------------------------------|
| `"register"`        | User registration process        | New account creation         |
| `"contact_number"`  | Phone number verification/update | Adding/updating phone number |
| `"email"`           | Email verification/update        | Adding/updating email address|
| `"forget_password"` | Password reset process           | Forgot password flow         |
| `"login"`           | Login verification               | Two-factor authentication    |

### Kotlin OtpChannel Enum (Reference)

```kotlin
enum class OtpChannel(val value: String) {
    REGISTER("register"),
    CONTACT_NUMBER("contact_number"),
    EMAIL("email"),
    FORGET_PASSWORD("forget_password"),
    LOGIN("login")
}
```

---

## OtpResponse Fields Reference

| Field Name | Description                     | Mandatory | Data Type |
|------------|---------------------------------|-----------|-----------|
| refcode    | Reference code for confirmation | O         | String    |
| expiredate | Expiration time (epoch millis)  | O         | Long/Int  |

---

## ConfirmOtpResponse Fields Reference

| Field Name | Description           | Mandatory | Data Type |
|------------|-----------------------|-----------|-----------|
| token      | Auth token on success | O         | String    |
| success    | Boolean status        | O         | Boolean   |

---

## ValidateOtpResponse Fields Reference

| Field Name   | Description           | Mandatory | Data Type |
|--------------|-----------------------|-----------|-----------|
| validatecode | Validation code/token | O         | String    |

---

### getOtp (Phone Number OTP)

Request OTP via SMS to phone number.

- Request (caller-supplied)

| Field Name    | Description                   | Mandatory | Data Type |
|---------------|-------------------------------|-----------|-----------|
| contactNumber | Phone number (E.164)          | M         | String    |
| channel       | OTP channel type (see above)  | M         | String    |

- Response `OtpResponse`
  HTTP status: 200

```kotlin
// Suspend
val result = otp.getOtp("+66900000000", "register")

// Callback
otp.getOtp("+66900000000", "register") { result ->
    when (result) {
        is OTPResult.SuccessGetOTP -> {
            // Handle successful OTP request
            val refCode = result.result.refcode
            val expireDate = result.result.expiredate
        }
        is OTPResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

#### Channel Usage Examples

```kotlin
// For user registration
otp.getOtp(contactNumber, "register") { result -> /* handle */ }

// For phone number verification/update
otp.getOtp(contactNumber, "contact_number") { result -> /* handle */ }

// For password reset
otp.getOtp(contactNumber, "forget_password") { result -> /* handle */ }

// For login 2FA
otp.getOtp(contactNumber, "login") { result -> /* handle */ }
```

---

### getOtpEmail (Email OTP)

Request OTP via email.

- Request (caller-supplied)

| Field Name | Description                  | Mandatory | Data Type |
|------------|------------------------------|-----------|-----------|
| email      | Email address                | M         | String    |
| channel    | OTP channel type (see above) | M         | String    |

- Response `OtpResponse`
  HTTP status: 200

```kotlin
// Suspend
val result = otp.getOtpEmail("user@example.com", "email")

// Callback
otp.getOtpEmail("user@example.com", "email") { result ->
    when (result) {
        is OTPResult.SuccessGetOTP -> {
            // Handle successful email OTP request
            val refCode = result.result.refcode
            val expireDate = result.result.expiredate
        }
        is OTPResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

#### Channel Usage Examples

```kotlin
// For user registration
otp.getOtpEmail(email, "register") { result -> /* handle */ }

// For email verification/update
otp.getOtpEmail(email, "email") { result -> /* handle */ }

// For password reset
otp.getOtpEmail(email, "forget_password") { result -> /* handle */ }

// For login 2FA
otp.getOtpEmail(email, "login") { result -> /* handle */ }
```

#### Error Handling (Both getOtp and getOtpEmail)

| Error Code | Error ID | Scenario               | User Message                                           | Recommended Action             |
|------------|----------|------------------------|--------------------------------------------------------|--------------------------------|
| 409        | 2078     | Contact already in use | "This contact is already in use. Please use another"   | Let user enter new contact     |
| 409        | 2092     | Invalid format         | "Invalid contact format"                               | Show correct format example    |
| 429        | -        | Rate limit exceeded    | "OTP requested too frequently. Please wait 60 seconds" | Show countdown timer           |
| 404        | -        | Contact not found      | "Contact not found in system"                          | Suggest user to register first |

#### Implementation Example

```kotlin
// Phone OTP
otp.getOtp(contactNumber, "register") { result ->
    when (result) {
        is OTPResult.SuccessGetOTP -> {
            startOtpTimer(result.result.expiredate)
        }
        is OTPResult.Error -> {
            handleOtpError(result.error)
        }
    }
}

// Email OTP
otp.getOtpEmail(email, "email") { result ->
    when (result) {
        is OTPResult.SuccessGetOTP -> {
            startOtpTimer(result.result.expiredate)
        }
        is OTPResult.Error -> {
            handleOtpError(result.error)
        }
    }
}

private fun handleOtpError(error: ErrorResponse) {
    val action = when {
        error.code == "409" && error.id == "2078" ->
            ErrorAction.ShowContactDuplicateError()
        error.code == "409" && error.id == "2092" ->
            ErrorAction.ShowContactFormatError()
        error.code == "429" ->
            ErrorAction.ShowRateLimitError(60)
        error.code == "404" ->
            ErrorAction.ShowContactNotFoundError()
        else ->
            ErrorAction.ShowGenericError(error.message)
    }
    handleErrorAction(action)
}
```

---

### confirmOtp

Confirm OTP and authenticate user. Returns auth token on success.

**Note:** Requires existing auth token (from device login).

- Request (caller-supplied)

| Field Name    | Description          | Mandatory | Data Type |
|---------------|----------------------|-----------|-----------|
| otp           | One-time password    | M         | String    |
| refCode       | Ref code from getOtp | M         | String    |
| contactNumber | Phone number (E.164) | M         | String    |

- Response `ConfirmOtpResponse`
  HTTP status: 200

```kotlin
// Suspend
val result = otp.confirmOtp(
    otp = "123456",
    refCode = "REFCODE",
    contactNumber = "+66900000000"
)

// Callback
otp.confirmOtp(
    otp = "123456",
    refCode = "REFCODE",
    contactNumber = "+66900000000"
) { result ->
    when (result) {
        is OTPResult.SuccessConfirmOTP -> {
            // Handle successful OTP confirmation
            val token = result.result.token
            val success = result.result.success
        }
        is OTPResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

#### Error Handling

| Error Code | Error ID | Scenario          | User Message                               | Recommended Action                                       |
|------------|----------|-------------------|--------------------------------------------|----------------------------------------------------------|
| 409        | 2091     | Invalid OTP       | "Invalid OTP code"                         | Let user enter OTP again, show "Request new code" button |
| 409        | 2092     | OTP expired       | "OTP code expired"                         | Resend OTP automatically                                 |
| 429        | -        | Too many attempts | "Too many attempts. Please wait 5 minutes" | Lock input and show countdown                            |

#### Implementation Example

```kotlin
otp.confirmOtp(otp, refCode, contactNumber) { result ->
    when (result) {
        is OTPResult.SuccessConfirmOTP -> {
            // Handle successful OTP confirmation
        }
        is OTPResult.Error -> {
            val action = when {
                result.error.code == "409" && result.error.id == "2091" ->
                    ErrorAction.ShowInvalidOtpError()
                result.error.code == "409" && result.error.id == "2092" ->
                    ErrorAction.ResendOtpAutomatically()
                result.error.code == "429" ->
                    ErrorAction.LockOtpInput(300) // 5 minutes
                else ->
                    ErrorAction.ShowGenericError(result.error.message)
            }
            handleErrorAction(action)
        }
    }
}
```

---

### validateOtp (Validate OTP for phone number)

Validate OTP without automatically logging in. Returns validation code.

- Request (caller-supplied)

| Field Name    | Description                  | Mandatory | Data Type |
|---------------|------------------------------|-----------|-----------|
| otp           | One-time password            | M         | String    |
| refCode       | Ref code from getOtp         | M         | String    |
| contactNumber | Phone number (E.164)         | M         | String    |
| use           | Use OTP immediately          | M         | Boolean   |
| channel       | OTP channel type (see above) | O         | String    |

- Response `ValidateOtpResponse`
  HTTP status: 200

```kotlin
// Suspend
val result = otp.validateOtp(
    otp = "123456",
    refCode = "REFCODE",
    contactNumber = "+66900000000",
    use = true,
    channel = "register"
)

// Callback
otp.validateOtp(
    otp = "123456",
    refCode = "REFCODE",
    contactNumber = "+66900000000",
    use = true,
    channel = "register"
) { result ->
    when (result) {
        is OTPResult.SuccessValidateOTP -> {
            // Handle successful OTP validation
            val validateCode = result.result.validatecode
        }
        is OTPResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

#### Difference from confirmOtp

| Method        | Purpose                                    | Returns           | Auto Login |
|---------------|--------------------------------------------|-------------------|------------|
| `validateOtp` | Validates OTP only                         | Validation code   | No         |
| `confirmOtp`  | Validates OTP AND authenticates user       | Auth token        | Yes        |

Use `validateOtp` when you need to verify OTP separately from login (e.g., registration flow), and `confirmOtp` for immediate authentication.

#### Error Handling

| Error Code | Error ID | Scenario          | User Message                               | Recommended Action                                       |
|------------|----------|-------------------|--------------------------------------------|----------------------------------------------------------|
| 409        | 2091     | Invalid OTP       | "Invalid OTP code"                         | Let user enter OTP again, show "Request new code" button |
| 409        | 2092     | OTP expired       | "OTP code expired"                         | Resend OTP automatically                                 |
| 429        | -        | Too many attempts | "Too many attempts. Please wait 5 minutes" | Lock input and show countdown                            |

---

### validateOtpEmail (Validate OTP for email)

Validate OTP sent to email without automatically logging in.

- Request (caller-supplied)

| Field Name | Description                  | Mandatory | Data Type |
|------------|------------------------------|-----------|-----------|
| otp        | One-time password            | M         | String    |
| refCode    | Ref code from getOtpEmail    | M         | String    |
| email      | Email address                | M         | String    |
| use        | Use OTP immediately          | M         | Boolean   |
| channel    | OTP channel type (see above) | O         | String    |

- Response `ValidateOtpResponse`
  HTTP status: 200

```kotlin
// Suspend
val result = otp.validateOtpEmail(
    otp = "123456",
    refCode = "REFCODE",
    email = "user@example.com",
    use = true,
    channel = "email"
)

// Callback
otp.validateOtpEmail(
    otp = "123456",
    refCode = "REFCODE",
    email = "user@example.com",
    use = true,
    channel = "email"
) { result ->
    when (result) {
        is OTPResult.SuccessValidateOTP -> {
            // Email verified successfully
            val validateCode = result.result.validatecode
        }
        is OTPResult.Error -> {
            val action = when {
                result.error.code == "409" && result.error.id == "2091" ->
                    ErrorAction.ShowInvalidOtpError()
                result.error.code == "409" && result.error.id == "2092" ->
                    ErrorAction.ResendOtpAutomatically()
                result.error.code == "429" ->
                    ErrorAction.LockOtpInput(300) // 5 minutes
                else ->
                    ErrorAction.ShowGenericError(result.error.message)
            }
            handleErrorAction(action)
        }
    }
}
```

---

## Complete OTP Flow Example

### Registration with Phone OTP

```kotlin
class RegistrationViewModel {
    private val otp = BuzzebeesSDK.instance().otp
    
    // Step 1: Request OTP
    fun requestOtp(phoneNumber: String) {
        otp.getOtp(phoneNumber, "register") { result ->
            when (result) {
                is OTPResult.SuccessGetOTP -> {
                    // Save refCode for later use
                    savedRefCode = result.result.refcode
                    // Start countdown timer
                    startTimer(result.result.expiredate)
                    // Navigate to OTP input screen
                    navigateToOtpInput()
                }
                is OTPResult.Error -> handleError(result.error)
            }
        }
    }
    
    // Step 2: Validate OTP
    fun verifyOtp(otpCode: String, phoneNumber: String) {
        otp.validateOtp(
            otp = otpCode,
            refCode = savedRefCode,
            contactNumber = phoneNumber,
            use = true,
            channel = "register"
        ) { result ->
            when (result) {
                is OTPResult.SuccessValidateOTP -> {
                    // OTP verified, proceed to complete registration
                    savedValidateCode = result.result.validatecode
                    navigateToRegistrationForm()
                }
                is OTPResult.Error -> handleError(result.error)
            }
        }
    }
}
```

### Login with OTP (2FA)

```kotlin
class LoginViewModel {
    private val otp = BuzzebeesSDK.instance().otp
    
    // Step 1: Request OTP for login
    fun requestLoginOtp(phoneNumber: String) {
        otp.getOtp(phoneNumber, "login") { result ->
            when (result) {
                is OTPResult.SuccessGetOTP -> {
                    savedRefCode = result.result.refcode
                    navigateToOtpInput()
                }
                is OTPResult.Error -> handleError(result.error)
            }
        }
    }
    
    // Step 2: Confirm OTP and login
    fun confirmLoginOtp(otpCode: String, phoneNumber: String) {
        otp.confirmOtp(
            otp = otpCode,
            refCode = savedRefCode,
            contactNumber = phoneNumber
        ) { result ->
            when (result) {
                is OTPResult.SuccessConfirmOTP -> {
                    // User is now logged in
                    // Token is automatically saved
                    navigateToHome()
                }
                is OTPResult.Error -> handleError(result.error)
            }
        }
    }
}
```

---
