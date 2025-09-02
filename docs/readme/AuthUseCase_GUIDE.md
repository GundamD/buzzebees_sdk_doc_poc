## AuthUseCase Guide

This guide shows how to initialize and use every public method in `AuthUseCase`, with suspend and
callback examples where available.

### Getting an instance

```kotlin
val auth = BuzzebeesSDK.instance().auth
```

---

### deviceLogin

- Request (caller-supplied only)

| Field Name | Description                                       | Mandatory | Data Type |
|------------|---------------------------------------------------|-----------|-----------|
| -          | None. SDK supplies device/app info automatically. | -         | -         |

- Response (`LoginResponse`)
  HTTP status: 200

> **All fields of `LoginResponse`**
>

## LoginResponse Fields Reference

| Field Name                      | Description                     | Mandatory | Data Type            |
|---------------------------------|---------------------------------|-----------|----------------------|
| token                           | Auth token                      | O         | String               |
| userId                          | Unique user id                  | O         | String               |
| appId                           | Application id                  | O         | String               |
| isFbUser                        | Logged in by Facebook           | O         | Boolean              |
| locale                          | Language code (e.g., 1054/1033) | O         | Int                  |
| userLevel                       | User level                      | O         | Int                  |
| userLevelDetail                 | Level detail                    | O         | String               |
| userFlags                       | User flags                      | O         | `Array<String>`        |
| sponsorId                       | Sponsor id                      | O         | Int                  |
| canRedeem                       | Redeem eligibility              | O         | Boolean              |
| uuid                            | Device/user uuid                | O         | String               |
| joinDate                        | Join date (epoch)               | O         | Int                  |
| usercode                        | User code                       | O         | String               |
| detail                          | Login detail info               | O         | LoginDetail          |
| detail.ewalletToken             | E-Wallet token                  | O         | String               |
| cartCount                       | Cart items count                | O         | Int                  |
| platform                        | Platform name                   | O         | String               |
| topics                          | Topics subscribed               | O         | `Array<String>`       |
| channel                         | Channel (device/facebook/web)   | O         | String               |
| TermAndCondition                | Terms & Conditions consent      | O         | Int                  |
| DataPrivacy                     | Data Privacy consent            | O         | Int                  |
| MarketingOption                 | Marketing option consent        | O         | Int                  |
| ConsentAge                      | Age consent                     | O         | Int                  |
| EmailMarketing                  | Email marketing opt-in          | O         | Int                  |
| SMSMarketing                    | SMS marketing opt-in            | O         | Int                  |
| NotificationMarketing           | Push notification opt-in        | O         | Int                  |
| LineMarketing                   | LINE marketing opt-in           | O         | Int                  |
| AnalyticsBuzzebeesCookies       | Analytics Buzzebees cookies     | O         | String               |
| AnalyticsFirebaseCookies        | Analytics Firebase cookies      | O         | String               |
| AnalyticsGoogleCookies          | Analytics Google cookies        | O         | String               |
| AnalyticsMetaCookies            | Analytics Meta cookies          | O         | String               |
| AnalyticsOtherCookies           | Other analytics cookies         | O         | String               |
| FunctionalCookies               | Functional cookies              | O         | String               |
| MarketingCookies                | Marketing cookies               | O         | String               |
| NecessaryCookies                | Necessary cookies               | O         | String               |
| jwt                             | JWT token                       | O         | String               |
| version                         | Version info                    | O         | Version              |
| version.allowUse                | Allow app usage                 | O         | Boolean              |
| version.hasNewVersion           | Has new version                 | O         | Boolean              |
| version.termandcondition        | Terms & Conditions text/flag    | O         | String               |
| version.dataprivacy             | Data privacy text/flag          | O         | String               |
| version.marketingoption         | Marketing option text/flag      | O         | String               |
| version.useCdn                  | Use CDN flag                    | O         | Boolean              |
| version.welcomePageTimes        | Welcome page count              | O         | Int                  |
| ewalletToken                    | E-Wallet token                  | O         | String               |
| authenticated                   | Authenticated status            | O         | Boolean              |
| sponsorPages                    | Sponsor pages link/info         | O         | String               |
| updatedPoints                   | Latest points info              | O         | UpdatedPoints        |
| updatedPoints.points            | User points                     | O         | Long                 |
| updatedPoints.time              | Last update timestamp           | O         | Long                 |
| account                         | Account info                    | O         | LoginAccountResponse |
| account.bzbsId                  | Buzzebees id                    | O         | String               |
| account.facebookId              | Facebook id                     | O         | String               |
| account.deviceId                | Device id                       | O         | String               |
| account.serverId                | Server id                       | O         | String               |
| account.lineUserId              | LINE user id                    | O         | String               |
| account.teamId                  | Team id                         | O         | String               |
| account.googleId                | Google id                       | O         | String               |
| account.appleId                 | Apple id                        | O         | String               |
| account.facebookMessengerUserId | Facebook Messenger user id      | O         | String               |
| account.adId                    | AD id                           | O         | String               |
| account.adB2CId                 | AD B2C id                       | O         | String               |
| account.zaloId                  | Zalo id                         | O         | String               |
| account.skipId                  | Skip id                         | O         | String               |

- Usage

```kotlin

// Suspend
val result = auth.deviceLoginSuspend()

// Callback
auth.loginDevice { result ->
    when (result) {
        is AuthResult.SuccessLogin -> {
            // Handle successful device login
            val token = result.result.token
            val userId = result.result.userId
            val userLevel = result.result.userLevel
            val cartCount = result.result.cartCount
        }
        is AuthResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}

```

---

### loginUsernamePassword

- Request (caller-supplied)

| Field Name | Description    | Mandatory | Data Type |
|------------|----------------|-----------|-----------|
| username   | Username/email | M         | String    |
| password   | Password       | M         | String    |

- Response: LoginResponse
  HTTP status: 200

> **All fields of `LoginResponse`** (same as in `deviceLogin` above)

- Usage

```kotlin
// Suspend
val result = auth.loginUsernamePassword("user@example.com", "password123")

// Callback
auth.loginUsernamePassword("user@example.com", "password123") { result ->
    when (result) {
        is AuthResult.SuccessLogin -> {
            // Handle successful username/password login
            val token = result.result.token
            val userId = result.result.userId
            val userLevel = result.result.userLevel
            val cartCount = result.result.cartCount
        }
        is AuthResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

#### Error Handling

| Error Code | Error ID | Scenario            | User Message                                         | Recommended Action                         |
|------------|----------|---------------------|------------------------------------------------------|--------------------------------------------|
| 401        | 401      | Invalid credentials | "Invalid password. Please try again"                 | Show error message, let user retry         |
| 403        | 403      | Account suspended   | "Account suspended. Please contact customer service" | Show support contact information           |
| 409        | 1417     | Account locked      | "Account locked"                                     | Show dialog with "Forgot Password?" button |
| 409        | 1905     | Account deactivated | "Account suspended. Please contact 02-105-1000"      | Show support phone number                  |
| 409        | 2076     | Session expired     | "Session expired. Please login again"                | Force logout and return to login screen    |

#### Implementation Example

```kotlin
auth.loginUsernamePassword(username, password) { result ->
    when (result) {
        is AuthResult.SuccessLogin -> {
            // Handle successful login
        }
        is AuthResult.Error -> {
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
            handleErrorAction(action)
        }
    }
}
```

---

## Auth header behavior

- Login methods (facebookLogin/googleLogin/lineLogin): no Authorization header is sent.
- Connect methods (connectFacebook/connectGoogle/connectLine): require Authorization header to
  perform a true connect. If missing, the server treats the call as a login.

---

### loginFacebook

- Request (caller-supplied)

| Field Name  | Description           | Mandatory | Data Type |
|-------------|-----------------------|-----------|-----------|
| accessToken | Facebook access token | M         | String    |

- Response: LoginResponse
  HTTP status: 200

> **All fields of `LoginResponse`** (same as in `deviceLogin`)

- Usage

```kotlin
// Suspend
val result = auth.loginFacebook("fb_access_token")

// Callback
auth.loginFacebook("fb_access_token") { result ->
    when (result) {
        is AuthResult.SuccessLogin -> {
            // Handle successful login
            val token = result.result.token
            val userId = result.result.userId
        }
        is AuthResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

Note: No prior token required.

---

### connectFacebook

- Request (caller-supplied)

| Field Name  | Description                     | Mandatory | Data Type |
|-------------|---------------------------------|-----------|-----------|
| accessToken | Link with Facebook access token | M         | String    |

- Response: LoginResponse
  HTTP status: 200

> **All fields of `LoginResponse`** (same as in `deviceLogin`)

- Usage

```kotlin
// Suspend
val result = auth.connectFacebook("fb_access_token")

// Callback
auth.connectFacebook("fb_access_token") { result ->
    when (result) {
        is AuthResult.SuccessLogin -> {
            // Handle successful connection
            val token = result.result.token
            val userId = result.result.userId
        }
        is AuthResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

Note: Requires existing auth token (SDK adds it to headers automatically).

---

### loginGoogle

- Request (caller-supplied)

| Field Name | Description     | Mandatory | Data Type |
|------------|-----------------|-----------|-----------|
| idToken    | Google ID token | M         | String    |

- Response: LoginResponse
  HTTP status: 200

> **All fields of `LoginResponse`** (same as in `deviceLogin`)

- Usage

```kotlin
// Suspend
val result = auth.loginGoogle("google_id_token")

// Callback
auth.loginGoogle("google_id_token") { result ->
    when (result) {
        is AuthResult.SuccessLogin -> {
            // Handle successful login
            val token = result.result.token
            val userId = result.result.userId
        }
        is AuthResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

Note: No prior token required.

---

### connectGoogle

- Request (caller-supplied)

| Field Name | Description               | Mandatory | Data Type |
|------------|---------------------------|-----------|-----------|
| idToken    | Link with Google ID token | M         | String    |

- Response: LoginResponse
  HTTP status: 200

> **All fields of `LoginResponse`** (same as in `deviceLogin`)

- Usage

```kotlin
// Suspend
val result = auth.connectGoogle("google_id_token")

// Callback
auth.connectGoogle("google_id_token") { result ->
    when (result) {
        is AuthResult.SuccessLogin -> {
            // Handle successful connection
            val token = result.result.token
            val userId = result.result.userId
        }
        is AuthResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

Note: Requires existing auth token (SDK adds it to headers automatically).

---

### loginLine

- Request (caller-supplied)

| Field Name      | Description       | Mandatory | Data Type |
|-----------------|-------------------|-----------|-----------|
| lineAccessToken | LINE access token | M         | String    |
| lineIdToken     | LINE ID token     | M         | String    |

- Response: LoginResponse
  HTTP status: 200

> **All fields of `LoginResponse`** (same as in `deviceLogin`)

- Usage

```kotlin
// Suspend
val result = auth.loginLine("line_access", "line_id_token")

// Callback
auth.loginLine("line_access", "line_id_token") { result ->
    when (result) {
        is AuthResult.SuccessLogin -> {
            // Handle successful login
            val token = result.result.token
            val userId = result.result.userId
        }
        is AuthResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

Note: No prior token required.

---

### connectLine

- Request (caller-supplied)

| Field Name      | Description                 | Mandatory | Data Type |
|-----------------|-----------------------------|-----------|-----------|
| lineAccessToken | Link with LINE access token | M         | String    |
| lineIdToken     | Link with LINE ID token     | M         | String    |

- Response: LoginResponse
  HTTP status: 200

> **All fields of `LoginResponse`** (same as in `deviceLogin`)

- Usage

```kotlin
// Suspend
val result = auth.connectLine("line_access", "line_id_token")

// Callback
auth.connectLine("line_access", "line_id_token") { result ->
    when (result) {
        is AuthResult.SuccessLogin -> {
            // Handle successful connection
            val token = result.result.token
            val userId = result.result.userId
        }
        is AuthResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

Note: Requires existing auth token (SDK adds it to headers automatically).

---

### logout

- Request (caller-supplied)

| Field Name | Description | Mandatory | Data Type |
|------------|-------------|-----------|-----------|
| -          | None        | -         | -         |

- Response  
  HTTP status: 204

| Type             | Description        |
|------------------|--------------------|
| SuccessNoContent | No body on success |

```kotlin
// Suspend
val result = auth.logout()

// Callback
auth.logout { result ->
    when (result) {
        is AuthResult.SuccessNoContent -> {
            // Handle successful logout
            // User is now logged out
        }
        is AuthResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### forgotPassword

- Request (caller-supplied)

| Field Name | Description   | Mandatory | Data Type |
|------------|---------------|-----------|-----------|
| email      | Account email | M         | String    |

- Response (`StatusResponse`)
  HTTP status: 200

| Field Name | Description             | Mandatory | Data Type |
|------------|-------------------------|-----------|-----------|
| status     | Operation status string | O         | String    |

```kotlin
// Suspend
val result = auth.forgotPassword("user@example.com")

// Callback
auth.forgotPassword("user@example.com") { result ->
    when (result) {
        is AuthResult.SuccessForGotPassword -> {
            // Handle successful password reset request
            val status = result.result.status
        }
        is AuthResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### resetPassword

- Request (caller-supplied)

| Field Name  | Description             | Mandatory | Data Type |
|-------------|-------------------------|-----------|-----------|
| email       | Account email           | M         | String    |
| refCode     | Reference code from OTP | M         | String    |
| newPassword | New password            | M         | String    |

- Response (`StatusResponse`)
  HTTP status: 200

| Field Name | Description             | Mandatory | Data Type |
|------------|-------------------------|-----------|-----------|
| status     | Operation status string | O         | String    |

```kotlin
// Suspend
val result = auth.resetPassword(
    email = "user@example.com",
    refCode = "REFCODE",
    newPassword = "NewP@ssw0rd"
)

// Callback
auth.resetPassword("user@example.com", "REFCODE", "NewP@ssw0rd") { result ->
    when (result) {
        is AuthResult.SuccessForGotPassword -> {
            // Handle successful password reset
            val status = result.result.status
        }
        is AuthResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### getOtp

- Request (caller-supplied)

| Field Name    | Description          | Mandatory | Data Type |
|---------------|----------------------|-----------|-----------|
| contactNumber | Phone number (E.164) | M         | String    |

- Response `OtpResponse`
  HTTP status: 200

| Field Name | Description                     | Mandatory | Data Type |
|------------|---------------------------------|-----------|-----------|
| refcode    | Reference code for confirmation | O         | String    |
| expiredate | Expiration time (epoch millis)  | O         | Long/Int  |

```kotlin
// Suspend
val result = auth.getOtp("+66900000000")

// Callback
auth.getOtp("+66900000000") { result ->
    when (result) {
        is AuthResult.SuccessGetOTP -> {
            // Handle successful OTP request
            val refCode = result.result.refcode
            val expireDate = result.result.expiredate
        }
        is AuthResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

#### Error Handling

| Error Code | Error ID | Scenario                | User Message                                               | Recommended Action                       |
|------------|----------|-------------------------|------------------------------------------------------------|------------------------------------------|
| 409        | 2078     | Mobile number duplicate | "This number is already in use. Please use another number" | Let user enter new number                |
| 409        | 2092     | Invalid mobile format   | "Invalid phone number format"                              | Show correct format example (08XXXXXXXX) |
| 429        | -        | Rate limit exceeded     | "OTP requested too frequently. Please wait 60 seconds"     | Show countdown timer                     |
| 404        | -        | Mobile not found        | "Phone number not found in system"                         | Suggest user to register first           |

#### Implementation Example

```kotlin
auth.getOtp(contactNumber) { result ->
    when (result) {
        is AuthResult.SuccessGetOTP -> {
            // Handle successful OTP request
            startOtpTimer(result.result.expiredate)
        }
        is AuthResult.Error -> {
            val action = when {
                result.error.code == "409" && result.error.id == "2078" ->
                    ErrorAction.ShowMobileDuplicateError()
                result.error.code == "409" && result.error.id == "2092" ->
                    ErrorAction.ShowMobileFormatError()
                result.error.code == "429" ->
                    ErrorAction.ShowRateLimitError(60) // 60 seconds
                result.error.code == "404" ->
                    ErrorAction.ShowMobileNotFoundError()
                else ->
                    ErrorAction.ShowGenericError(result.error.message)
            }
            handleErrorAction(action)
        }
    }
}
```

---

### confirmOtp

- Request (caller-supplied)

| Field Name    | Description          | Mandatory | Data Type |
|---------------|----------------------|-----------|-----------|
| otp           | One-time password    | M         | String    |
| refCode       | Ref code from getOtp | M         | String    |
| contactNumber | Phone number (E.164) | M         | String    |

- Response `ConfirmOtpResponse`
  HTTP status: 200

| Field Name | Description           | Mandatory | Data Type |
|------------|-----------------------|-----------|-----------|
| token      | Auth token on success | O         | String    |
| success    | Boolean status        | O         | Boolean   |

```kotlin
// Suspend
val result = auth.confirmOtp(
    otp = "123456",
    refCode = "REFCODE",
    contactNumber = "+66900000000"
)

// Callback
auth.confirmOtp(
    otp = "123456",
    refCode = "REFCODE",
    contactNumber = "+66900000000"
) { result ->
    when (result) {
        is AuthResult.SuccessConfirmOTP -> {
            // Handle successful OTP confirmation
            val token = result.result.token
            val success = result.result.success
        }
        is AuthResult.Error -> {
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
auth.confirmOtp(otp, refCode, contactNumber) { result ->
    when (result) {
        is AuthResult.SuccessConfirmOTP -> {
            // Handle successful OTP confirmation
        }
        is AuthResult.Error -> {
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

### resume (token refresh via backend)

- Request (caller-supplied)

| Field Name | Description | Mandatory | Data Type |
|------------|-------------|-----------|-----------|
| -          | None        | -         | -         |

- Response (`ResumeResponse`)  
  HTTP status: 200

| Field Name      | Description           | Mandatory | Data Type |
|-----------------|-----------------------|-----------|-----------|
| userId          | User id               | O         | String    |
| appId           | Application id        | O         | String    |
| isFbUser        | Logged in by Facebook | O         | Boolean   |
| locale          | Language code         | O         | Int       |
| userLevel       | User level            | O         | Int       |
| userLevelDetail | Level detail          | O         | String    |
| uuid            | Device/User uuid      | O         | String    |
| cartCount       | Cart items count      | O         | Int       |
| channel         | Channel               | O         | String    |
| version         | Version info          | O         | Version   |
| token           | Auth token            | O         | String    |
| jwt             | JWT token             | O         | String    |

```kotlin
// Suspend
val result = auth.resume()

// Callback
auth.resume { result ->
    when (result) {
        is AuthResult.SuccessResume -> {
            // Handle successful resume
            val token = result.result.token
            val userId = result.result.userId
            val userLevel = result.result.userLevel
        }
        is AuthResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### updateDevice (send latest device info/FCM)

- Request (caller-supplied)

| Field Name | Description | Mandatory | Data Type |
|------------|-------------|-----------|-----------|
| -          | None        | -         | -         |

- Response  
  HTTP status: 204

| Type             | Description        |
|------------------|--------------------|
| SuccessNoContent | No body on success |

```kotlin
// Suspend
val result = auth.updateDevice()

// Callback
auth.updateDevice { result ->
    when (result) {
        is AuthResult.SuccessNoContent -> {
            // Handle successful device update
            // Device info and FCM token updated
        }
        is AuthResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### checkVersion

- Request (caller-supplied)

| Field Name | Description | Mandatory | Data Type |
|------------|-------------|-----------|-----------|
| -          | None        | -         | -         |

- Response (`Version`)
  HTTP status: 200

| Field Name       | Description                  | Mandatory | Data Type |
|------------------|------------------------------|-----------|-----------|
| allowUse         | Allow application usage      | O         | Boolean   |
| hasNewVersion    | Has new app version          | O         | Boolean   |
| termandcondition | Terms & Conditions text/flag | O         | String    |
| dataprivacy      | Data privacy text/flag       | O         | String    |
| marketingoption  | Marketing option text/flag   | O         | String    |
| useCdn           | Use CDN flag                 | O         | Boolean   |
| welcomePageTimes | Welcome page count           | O         | Int       |

```kotlin
// Suspend
val result = auth.checkVersion()

// Callback
auth.checkVersion { result ->
    when (result) {
        is AuthResult.SuccessCheckVersion -> {
            // Handle successful version check
            val allowUse = result.result.allowUse
            val hasNewVersion = result.result.hasNewVersion
        }
        is AuthResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### fraud (device risk check)

- Request (caller-supplied)

| Field Name | Description | Mandatory | Data Type |
|------------|-------------|-----------|-----------|
| -          | None        | -         | -         |

- Response  
  HTTP status: 200

| Type     | Description          |
|----------|----------------------|
| Raw body | okhttp3.ResponseBody |

```kotlin
// Suspend
val result = auth.fraud()

// Callback
auth.fraud { result ->
    when (result) {
        is AuthResult.SuccessFraud -> {
            // Handle successful fraud check
            val responseBody = result.result
            // Process response body as needed
        }
        is AuthResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```
