## ConsentUseCase Guide

This guide shows how to initialize and use every public method in `ConsentUseCase`, with suspend
and callback examples where available. The ConsentUseCase provides comprehensive user consent
management functionality including updating consent preferences, retrieving current consent status,
and revoking user consent.

### Getting an instance

```kotlin
val consentService = BuzzebeesSDK.instance().consentUseCase
```

---

### updateConsent

Updates user consent preferences for various marketing channels and policy versions.

- Request (caller-supplied)

| Field Name | Description                        | Mandatory | Data Type   |
|------------|------------------------------------|-----------|-------------|
| form       | Consent form with user preferences | M         | ConsentForm |

#### ConsentForm Fields

| Field Name              | Description                                     | Mandatory | Data Type           |
|-------------------------|-------------------------------------------------|-----------|---------------------|
| termAndConditionVersion | Version of terms and conditions accepted        | O         | Int?                |
| dataPrivacyVersion      | Version of data privacy policy accepted         | O         | Int?                |
| marketingOptionsVersion | Version of marketing options accepted           | O         | Int?                |
| emailMarketing          | Email marketing consent (0 = false, 1 = true)   | O         | Int?                |
| smsMarketing            | SMS marketing consent (0 = false, 1 = true)     | O         | Int?                |
| notificationMarketing   | Push notification consent (0 = false, 1 = true) | O         | Int?                |
| lineMarketing           | LINE messaging consent (0 = false, 1 = true)    | O         | Int?                |
| consentAge              | User age for consent validation                 | O         | Int?                |
| options                 | Additional consent options                      | O         | Map<String, String> |

- Response (`Consent`) [source](../buzzebees_sdk/src/main/java/com/buzzebees/sdk/entity/consent/Consent.kt)  
  HTTP status: 200

### Consent Entity Fields

| Field Name            | Description                      | Data Type | JSON Field            |
|-----------------------|----------------------------------|-----------|-----------------------|
| dataPrivacy           | Data privacy policy version      | Int?      | DataPrivacy           |
| marketingOption       | Marketing options version        | Int?      | MarketingOption       |
| termAndCondition      | Terms and conditions version     | Int?      | TermAndCondition      |
| consentAge            | User consent age                 | Int?      | ConsentAge            |
| lineMarketing         | LINE marketing consent status    | Int?      | LineMarketing         |
| notificationMarketing | Push notification consent status | Int?      | NotificationMarketing |
| smsMarketing          | SMS marketing consent status     | Int?      | SMSMarketing          |
| emailMarketing        | Email marketing consent status   | Int?      | EmailMarketing        |

- Usage

```kotlin
// Create consent form
val consentForm = ConsentForm(
    termAndConditionVersion = 1,
    dataPrivacyVersion = 2,
    marketingOptionsVersion = 1,
    emailMarketing = 1, // Agree to email marketing
    smsMarketing = 0,   // Decline SMS marketing
    notificationMarketing = 1, // Agree to push notifications
    lineMarketing = 0,  // Decline LINE marketing
    consentAge = 25,
    options = mapOf("customOption" to "value")
)

// Suspend
val result = consentService.updateConsent(consentForm)

// Callback
consentService.updateConsent(consentForm) { result ->
    when (result) {
        is ConsentResult.SuccessUpdateConsent -> {
            // Handle successful consent update
            val updatedConsent = result.result
            val emailConsent = updatedConsent.emailMarketing
            val smsConsent = updatedConsent.smsMarketing
            val notificationConsent = updatedConsent.notificationMarketing
            val lineConsent = updatedConsent.lineMarketing

            println("Consent updated successfully!")
            println("Email marketing: ${if (emailConsent == 1) "Enabled" else "Disabled"}")
            println("SMS marketing: ${if (smsConsent == 1) "Enabled" else "Disabled"}")
            println("Push notifications: ${if (notificationConsent == 1) "Enabled" else "Disabled"}")
            println("LINE marketing: ${if (lineConsent == 1) "Enabled" else "Disabled"}")
        }
        is ConsentResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message

            when (errorCode) {
                "INVALID_CONSENT_AGE" -> {
                    // Handle invalid age for consent
                }
                "CONSENT_VERSION_MISMATCH" -> {
                    // Handle version mismatch
                }
                "REQUIRED_CONSENT_MISSING" -> {
                    // Handle missing required consent
                }
            }
        }
    }
}
```

---

### getConsent

Retrieves the current user consent status and preferences.

- Request (caller-supplied)

No parameters required.

- Response (`Consent`) [source](../buzzebees_sdk/src/main/java/com/buzzebees/sdk/entity/consent/Consent.kt)  
  HTTP status: 200

### Consent Entity Fields

| Field Name            | Description                      | Data Type | JSON Field            |
|-----------------------|----------------------------------|-----------|-----------------------|
| dataPrivacy           | Data privacy policy version      | Int?      | DataPrivacy           |
| marketingOption       | Marketing options version        | Int?      | MarketingOption       |
| termAndCondition      | Terms and conditions version     | Int?      | TermAndCondition      |
| consentAge            | User consent age                 | Int?      | ConsentAge            |
| lineMarketing         | LINE marketing consent status    | Int?      | LineMarketing         |
| notificationMarketing | Push notification consent status | Int?      | NotificationMarketing |
| smsMarketing          | SMS marketing consent status     | Int?      | SMSMarketing          |
| emailMarketing        | Email marketing consent status   | Int?      | EmailMarketing        |

- Usage

```kotlin
// Suspend
val result = consentService.getConsent()

// Callback
consentService.getConsent { result ->
    when (result) {
        is ConsentResult.SuccessConsent -> {
            // Handle successful consent retrieval
            val consent = result.result
            val termVersion = consent.termAndCondition
            val privacyVersion = consent.dataPrivacy
            val marketingVersion = consent.marketingOption
            val userAge = consent.consentAge

            println("Current consent status:")
            println("Terms & Conditions version: $termVersion")
            println("Data Privacy version: $privacyVersion")
            println("Marketing Options version: $marketingVersion")
            println("User consent age: $userAge")

            // Check marketing preferences
            val emailEnabled = consent.emailMarketing == 1
            val smsEnabled = consent.smsMarketing == 1
            val notificationEnabled = consent.notificationMarketing == 1
            val lineEnabled = consent.lineMarketing == 1

            println("Marketing Preferences:")
            println("Email: ${if (emailEnabled) "Enabled" else "Disabled"}")
            println("SMS: ${if (smsEnabled) "Enabled" else "Disabled"}")
            println("Notifications: ${if (notificationEnabled) "Enabled" else "Disabled"}")
            println("LINE: ${if (lineEnabled) "Enabled" else "Disabled"}")
        }
        is ConsentResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message

            when (errorCode) {
                "USER_NOT_FOUND" -> {
                    // Handle user not found
                }
                "CONSENT_NOT_INITIALIZED" -> {
                    // Handle consent not yet initialized
                }
            }
        }
    }
}
```

---

### unConsent

Revokes all user consent and opt-outs from all marketing communications.

- Request (caller-supplied)

No parameters required.

- Response (`Consent`) [source](../buzzebees_sdk/src/main/java/com/buzzebees/sdk/entity/consent/Consent.kt)  
  HTTP status: 200

### Consent Entity Fields

| Field Name            | Description                      | Data Type | JSON Field            |
|-----------------------|----------------------------------|-----------|-----------------------|
| dataPrivacy           | Data privacy policy version      | Int?      | DataPrivacy           |
| marketingOption       | Marketing options version        | Int?      | MarketingOption       |
| termAndCondition      | Terms and conditions version     | Int?      | TermAndCondition      |
| consentAge            | User consent age                 | Int?      | ConsentAge            |
| lineMarketing         | LINE marketing consent status    | Int?      | LineMarketing         |
| notificationMarketing | Push notification consent status | Int?      | NotificationMarketing |
| smsMarketing          | SMS marketing consent status     | Int?      | SMSMarketing          |
| emailMarketing        | Email marketing consent status   | Int?      | EmailMarketing        |

- Usage

```kotlin
// Suspend
val result = consentService.unConsent()

// Callback
consentService.unConsent { result ->
    when (result) {
        is ConsentResult.SuccessUnConsent -> {
            // Handle successful consent revocation
            val revokedConsent = result.result

            println("User consent revoked successfully!")
            println("All marketing communications have been disabled")

            // Verify all marketing preferences are disabled
            val emailDisabled = revokedConsent.emailMarketing == 0
            val smsDisabled = revokedConsent.smsMarketing == 0
            val notificationDisabled = revokedConsent.notificationMarketing == 0
            val lineDisabled = revokedConsent.lineMarketing == 0

            if (emailDisabled && smsDisabled && notificationDisabled && lineDisabled) {
                println("All marketing channels successfully disabled")
            }
        }
        is ConsentResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message

            when (errorCode) {
                "CONSENT_ALREADY_REVOKED" -> {
                    // Handle already revoked consent
                }
                "REVOCATION_NOT_ALLOWED" -> {
                    // Handle revocation not permitted
                }
            }
        }
    }
}
```

---

## Summary

The ConsentUseCase provides comprehensive user consent management functionality for privacy
compliance and marketing preferences within the Buzzebees SDK. It offers granular control over
different communication channels and policy versions, making it essential for GDPR, CCPA, and other
privacy regulation compliance in applications.