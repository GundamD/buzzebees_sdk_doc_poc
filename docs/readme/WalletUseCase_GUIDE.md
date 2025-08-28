## WalletUseCase Guide

This guide shows how to initialize and use every public method in `WalletUseCase`, with suspend and callback examples where available. The WalletUseCase provides digital wallet functionality for managing stamp codes and earning codes, enabling users to generate time-sensitive codes for stamp collection and reward earning activities.

### Getting an instance

```kotlin
val walletService = BuzzebeesSDK.instance().walletUseCase
```

---

### getStampCode

Generates a time-sensitive stamp code that can be used for stamp collection activities. This code is typically used at point-of-sale or check-in locations to collect stamps.

- Request (caller-supplied)

| Field Name  | Description                    | Mandatory | Data Type |
|-------------|--------------------------------|-----------|-----------|
| macAddress  | Device MAC address (optional)  | O         | String?   |
| cardId      | Stamp card identifier          | M         | String    |

- Response (`StampCode`
  HTTP status: 200

### StampCode Entity Fields

| Field Name  | Description                 | Data Type | JSON Field   |
|-------------|-----------------------------|-----------|--------------|
| result      | Operation result status     | String?   | result       |
| code        | Generated stamp code        | String?   | code         |
| expirein    | Code expiration timestamp   | Long?     | expirein     |
| codeFormat  | Code format specification   | String?   | code_format  |

- Usage

```kotlin
// Suspend
val result = walletService.getStampCode(
    macAddress = "AA:BB:CC:DD:EE:FF",
    cardId = "card_12345"
)

// Callback
walletService.getStampCode("AA:BB:CC:DD:EE:FF", "card_12345") { result ->
    when (result) {
        is WalletResult.SuccessStampCode -> {
            // Handle successful stamp code generation
            val stampCode = result.result
            
            println("Stamp code generated: ${stampCode.code}")
            println("Expires at: ${java.util.Date(stampCode.expirein ?: 0)}")
            println("Format: ${stampCode.codeFormat}")
        }
        is WalletResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
            
            when (errorCode) {
                "INVALID_CARD_ID" -> {
                    println("Invalid stamp card ID")
                }
                "CARD_EXPIRED" -> {
                    println("Stamp card has expired")
                }
                else -> {
                    println("Error: $errorMessage")
                }
            }
        }
    }
}
```

---

### earnCode

Generates a time-sensitive earning code that can be used to earn rewards, points, or benefits. This code is typically used for promotional activities or reward collection.

- Request (caller-supplied)

| Field Name   | Description                     | Mandatory | Data Type |
|--------------|---------------------------------|-----------|-----------|
| macAddress   | Device MAC address (optional)   | O         | String?   |
| deviceAppId  | Device application ID (optional)| O         | String?   |
| agencyId     | Agency identifier               | M         | String    |

- Response (`EarnCode`) 
  HTTP status: 200

### EarnCode Entity Fields

| Field Name  | Description                 | Data Type | JSON Field   |
|-------------|-----------------------------|-----------|--------------|
| result      | Operation result status     | String?   | result       |
| code        | Generated earning code      | String?   | code         |
| expirein    | Code expiration timestamp   | Long?     | expirein     |
| codeFormat  | Code format specification   | String?   | code_format  |

- Usage

```kotlin
// Suspend
val result = walletService.earnCode(
    macAddress = "AA:BB:CC:DD:EE:FF",
    deviceAppId = "app_54321",
    agencyId = "agency_789"
)

// Callback
walletService.earnCode("AA:BB:CC:DD:EE:FF", "app_54321", "agency_789") { result ->
    when (result) {
        is WalletResult.SuccessEarnCode -> {
            // Handle successful earning code generation
            val earnCode = result.result
            
            println("Earning code generated: ${earnCode.code}")
            println("Expires at: ${java.util.Date(earnCode.expirein ?: 0)}")
            println("Format: ${earnCode.codeFormat}")
        }
        is WalletResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
            
            when (errorCode) {
                "INVALID_AGENCY_ID" -> {
                    println("Invalid agency ID")
                }
                "CAMPAIGN_ENDED" -> {
                    println("Campaign has ended")
                }
                else -> {
                    println("Error: $errorMessage")
                }
            }
        }
    }
}
```

---

## Summary

The WalletUseCase provides digital wallet functionality within the Buzzebees SDK. It offers two main methods:

- **getStampCode**: Generate time-sensitive codes for stamp collection activities
- **earnCode**: Generate time-sensitive codes for earning rewards and benefits

Both methods support suspend and callback patterns with comprehensive error handling and time-sensitive code generation for secure wallet operations.
