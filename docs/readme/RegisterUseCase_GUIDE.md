## RegisterUseCase Guide

This guide shows how to initialize and use every public method in `RegisterUseCase`, with suspend
and callback examples where available. The RegisterUseCase provides user registration functionality
for creating new user accounts with comprehensive profile information and marketing preferences.

### Getting an instance

```kotlin
val registerService = BuzzebeesSDK.instance().registerUseCase
```

---

### register

Registers a new user account with comprehensive profile information and marketing preferences.

- Request (caller-supplied)

| Field Name               | Description                            | Mandatory | Data Type           |
|--------------------------|----------------------------------------|-----------|---------------------|
| appId                    | Application identifier                 | M         | String              |
| uuid                     | Device UUID                            | M         | String              |
| macAddress               | Device MAC address                     | M         | String              |
| os                       | Operating system                       | M         | String              |
| platform                 | Platform type                          | M         | String              |
| clientVersion            | Client application version             | M         | String              |
| deviceNotificationEnable | Device notification enabled flag       | M         | Boolean             |
| username                 | Username for login                     | M         | String              |
| password                 | User password                          | M         | String              |
| confirmPassword          | Password confirmation                  | M         | String              |
| firstname                | First name                             | M         | String              |
| lastname                 | Last name                              | M         | String              |
| address                  | User address                           | O         | String?             |
| gender                   | Gender                                 | O         | String?             |
| birthdate                | Birth date timestamp                   | O         | Long?               |
| email                    | Email address                          | O         | String?             |
| contact_number           | Contact phone number                   | M         | String              |
| otp                      | OTP verification code                  | M         | String              |
| refCode                  | Reference code for OTP                 | M         | String              |
| refUserCode              | Referral user code                     | O         | String?             |
| termAndConditionVersion  | Terms and conditions version           | O         | String?             |
| dataPrivacyVersion       | Data privacy version                   | O         | String?             |
| marketingOptionsVersion  | Marketing options version              | O         | String?             |
| emailMarketing           | Email marketing preference             | O         | Int?                |
| smsMarketing             | SMS marketing preference               | O         | Int?                |
| notificationMarketing    | Push notification marketing preference | O         | Int?                |
| lineMarketing            | LINE marketing preference              | O         | Int?                |
| phoneMarketing           | Phone marketing preference             | O         | Int?                |
| options                  | Additional options                     | O         | Map<String, String> |

- Response (`RegisterResponse`) [source](../buzzebees_sdk/src/main/java/com/buzzebees/sdk/entity/register/RegisterResponse.kt)  
  HTTP status: 200

### RegisterResponse Entity Fields

| Field Name    | Description          | Data Type | JSON Field     |
|---------------|----------------------|-----------|----------------|
| status        | Registration status  | String?   | status         |
| username      | Registered username  | String?   | username       |
| userId        | Generated user ID    | String?   | userId         |
| firstname     | First name           | String?   | firstname      |
| lastname      | Last name            | String?   | lastname       |
| address       | User address         | String?   | address        |
| email         | Email address        | String?   | email          |
| clientVersion | Client version used  | String?   | client_version |
| carrier       | Mobile carrier       | String?   | carrier        |
| macAddress    | Device MAC address   | String?   | mac_address    |
| platform      | Platform type        | String?   | platform       |
| os            | Operating system     | String?   | os             |
| gender        | Gender               | String?   | gender         |
| birthdate     | Birth date timestamp | Long?     | birthdate      |

- Usage

```kotlin
// Create registration form
val registerForm = RegisterForm(
    appId = "my_app_id",
    uuid = "device_uuid_12345",
    macAddress = "AA:BB:CC:DD:EE:FF",
    os = "Android",
    platform = "mobile",
    clientVersion = "1.0.0",
    deviceNotificationEnable = true,
    username = "john_doe",
    password = "securePassword123",
    confirmPassword = "securePassword123",
    firstname = "John",
    lastname = "Doe",
    address = "123 Main Street, Bangkok",
    gender = "Male",
    birthdate = 631152000000L, // January 1, 1990
    email = "john.doe@example.com",
    contact_number = "0812345678",
    otp = "123456",
    refCode = "REF123",
    refUserCode = "REFERRAL456",
    termAndConditionVersion = "1.0",
    dataPrivacyVersion = "1.0",
    marketingOptionsVersion = "1.0",
    emailMarketing = 1, // 1 = enabled, 0 = disabled
    smsMarketing = 1,
    notificationMarketing = 1,
    lineMarketing = 0,
    phoneMarketing = 0,
    options = mapOf(
        "source" to "mobile_app",
        "campaign" to "summer_2024"
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

            val status = registerResponse.status
            val userId = registerResponse.userId
            val username = registerResponse.username
            val firstname = registerResponse.firstname
            val lastname = registerResponse.lastname
            val email = registerResponse.email

            println("Registration successful!")
            println("Status: $status")
            println("User ID: $userId")
            println("Username: $username")
            println("Name: $firstname $lastname")
            println("Email: $email")
        }
        is RegisterResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message

            when (errorCode) {
                "USERNAME_EXISTS" -> {
                    // Handle username already exists
                    println("Username already exists")
                }
                "EMAIL_EXISTS" -> {
                    // Handle email already exists
                    println("Email already registered")
                }
                "INVALID_OTP" -> {
                    // Handle invalid OTP
                    println("Invalid OTP code")
                }
                "PASSWORD_MISMATCH" -> {
                    // Handle password confirmation mismatch
                    println("Passwords do not match")
                }
                "INVALID_PHONE" -> {
                    // Handle invalid phone number
                    println("Invalid phone number format")
                }
            }
        }
    }
}
```

---

## Summary

The RegisterUseCase provides comprehensive user registration functionality within the Buzzebees SDK.
It offers a single method to register new users with extensive profile information including device
details, personal information, marketing preferences, and referral codes. The registration process
includes OTP verification for phone numbers and supports various marketing consent options for
different communication channels.
