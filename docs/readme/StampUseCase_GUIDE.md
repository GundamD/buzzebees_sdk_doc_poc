## StampUseCase Guide

This guide shows how to initialize and use every public method in `StampUseCase`, with suspend
and callback examples where available. The StampUseCase provides comprehensive stamp-based loyalty program
functionality for creating stamps, managing stamp collections, and tracking loyalty program progress.

### Getting an instance

```kotlin
val stampService = BuzzebeesSDK.instance().stampUseCase
```

---

### createStamp

Creates a new stamp for loyalty program participation.

- Request (caller-supplied)

| Field Name | Description             | Mandatory | Data Type |
|------------|-------------------------|-----------|-----------|
| issuer     | Issuer identifier       | M         | String    |

- Response (`CreateStampResponse`) 
  HTTP status: 200

### CreateStampResponse Entity Fields

| Field Name | Description              | Data Type | JSON Field |
|------------|--------------------------|-----------|------------|
| success    | Creation success flag    | Boolean?  | success    |
| balance    | Account balance          | Double?   | balance    |
| cardId     | Card identifier          | String?   | cardId     |
| userId     | User identifier          | String?   | userId     |
| imei       | Device IMEI              | String?   | imei       |
| appId      | Application identifier   | String?   | app_id     |
| issuer     | Issuer identifier        | String?   | issuer     |
| stampId    | Created stamp identifier | String?   | stampId    |
| sponsorId  | Sponsor identifier       | Int?      | sponsorId  |

- Usage

```kotlin
// Suspend
val result = stampService.createStamp("LOYALTY_PARTNER_001")

// Callback
stampService.createStamp("LOYALTY_PARTNER_001") { result ->
    when (result) {
        is StampResult.SuccessCreate -> {
            // Handle successful stamp creation
            val createResponse = result.result
            println("Stamp created successfully!")
            println("Stamp ID: ${createResponse.stampId}")
            println("Card ID: ${createResponse.cardId}")
            println("Balance: ${createResponse.balance}")
        }
        is StampResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### getStampList

Retrieves a list of all user's stamps in the loyalty program.

- Request (caller-supplied)

No parameters required.

- Response (`List<Stamp>`)  
  HTTP status: 200

### Stamp Entity Fields

| Field Name         | Description                 | Data Type | JSON Field         |
|--------------------|-----------------------------|-----------|--------------------|
| amount             | Stamp amount                | Double?   | amount             |
| agencyId           | Agency identifier           | Int?      | agencyId           |
| cardId             | Card identifier             | String?   | cardId             |
| issuer             | Issuer identifier           | String?   | issuer             |
| stampId            | Stamp identifier            | String?   | stampId            |
| imageUrl           | Stamp image URL             | String?   | imageUrl           |
| owner              | Stamp owner                 | String?   | owner              |
| timestamp          | Creation timestamp          | Int?      | timestamp          |
| active             | Active status flag          | Boolean?  | active             |
| name               | Stamp name                  | String?   | name               |
| description        | Stamp description           | String?   | description        |
| stampImageUrl      | Stamp image URL             | String?   | stampImageUrl      |
| maxQuantity        | Maximum stamp quantity      | Int?      | maxQuantity        |
| currentQuantity    | Current stamp quantity      | Int?      | currentQuantity    |
| pricePerStamp      | Price per stamp             | Double?   | pricePerStamp      |
| codeExpireIn       | Code expiration time        | Int?      | codeExpireIn       |
| stampRawScore      | Raw stamp score             | Int?      | stampRawScore      |
| stampScore         | Stamp score                 | Int?      | stampScore         |
| stampSpecialScore  | Special stamp score         | Int?      | stampSpecialScore  |

- Usage

```kotlin
// Suspend
val result = stampService.getStampList()

// Callback
stampService.getStampList { result ->
    when (result) {
        is StampResult.SuccessStamp -> {
            // Handle successful stamp list retrieval
            val stamps = result.result
            stamps.forEach { stamp ->
                println("Stamp: ${stamp.name}")
                println("Progress: ${stamp.currentQuantity}/${stamp.maxQuantity}")
                println("Price per stamp: ${stamp.pricePerStamp}")
            }
        }
        is StampResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### getStampProfile

Retrieves detailed stamp profile information for a specific stamp and card.

- Request (caller-supplied)

| Field Name | Description       | Mandatory | Data Type |
|------------|-------------------|-----------|-----------|
| stampId    | Stamp identifier  | M         | String    |
| cardId     | Card identifier   | M         | String    |

- Response (`StampProfileResponse`)
  HTTP status: 200

### StampProfileResponse Entity Fields

| Field Name        | Description               | Data Type            | JSON Field        |
|-------------------|---------------------------|----------------------|-------------------|
| id                | Profile identifier        | String?              | id                |
| agencyId          | Agency identifier         | Int?                 | agencyId          |
| cardId            | Card identifier           | String?              | cardId            |
| name              | Stamp name                | String?              | name              |
| description       | Stamp description         | String?              | description       |
| imageUrl          | Stamp image URL           | String?              | imageUrl          |
| backgroundUrl     | Background image URL      | String?              | backgroundUrl     |
| maxQuantity       | Maximum quantity          | Int?                 | maxQuantity       |
| currentQuantity   | Current quantity          | Int?                 | currentQuantity   |
| expireDate        | Expiration date           | Long?                | expireDate        |
| pricePerStamp     | Price per stamp           | Double?              | pricePerStamp     |
| otherPromotions   | Other promotions data     | Any?                 | otherPromotions   |
| otherStamps       | List of other stamps      | `List<Stamp>?`         | otherStamps       |
| campaigns         | List of campaigns         | `List<StampCampaign>?` | campaigns         |
| history           | Stamp history             | `List<StampHistory>?`  | history           |

### StampCampaign Entity Fields

| Field Name | Description         | Data Type | JSON Field |
|------------|---------------------|-----------|------------|
| id         | Campaign identifier | Int?      | id         |
| imgUrl     | Campaign image URL  | String?   | img_url    |
| qty        | Campaign quantity   | Int?      | qty        |

### StampHistory Entity Fields

| Field Name     | Description              | Data Type | JSON Field     |
|----------------|--------------------------|-----------|----------------|
| amount         | Transaction amount       | Double?   | amount         |
| terminalId     | Terminal identifier      | String?   | terminalId     |
| branchId       | Branch identifier        | String?   | branchId       |
| branchName     | Branch name              | String?   | branchName     |
| brandId        | Brand identifier         | String?   | brandId        |
| customerId     | Customer identifier      | String?   | customerId     |
| customerName   | Customer name            | Any?      | customerName   |
| campaignId     | Campaign identifier      | Any?      | campaignId     |
| description    | Transaction description  | String?   | description    |
| issuer         | Issuer identifier        | String?   | issuer         |
| issuerName     | Issuer name              | String?   | issuerName     |
| imageUrl       | Transaction image URL    | String?   | imageUrl       |
| merchant       | Merchant identifier      | String?   | merchant       |
| merchantName   | Merchant name            | String?   | merchantName   |
| status         | Transaction status       | Int?      | status         |
| timestamp      | Transaction timestamp    | Int?      | timestamp      |
| type           | Transaction type         | String?   | type           |
| stampCount     | Number of stamps earned  | Int?      | stampCount     |
| createDate     | Creation date            | Long?     | createDate     |
| name           | Transaction name         | String?   | name           |
| backgroundUrl  | Background image URL     | String?   | backgroundUrl  |

- Usage

```kotlin
// Suspend
val result = stampService.getStampProfile("stamp_123", "card_456")

// Callback
stampService.getStampProfile("stamp_123", "card_456") { result ->
    when (result) {
        is StampResult.SuccessStampProfile -> {
            // Handle successful stamp profile retrieval
            val profile = result.result
            println("Stamp Profile: ${profile.name}")
            println("Description: ${profile.description}")
            println("Progress: ${profile.currentQuantity}/${profile.maxQuantity}")
            println("Expires: ${profile.expireDate}")
            
            // Show campaigns
            profile.campaigns?.forEach { campaign ->
                println("Campaign: ${campaign.id}, Quantity: ${campaign.qty}")
            }
            
            // Show history
            profile.history?.forEach { history ->
                println("Transaction: ${history.description}, Stamps: ${history.stampCount}")
            }
        }
        is StampResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

## Summary

The StampUseCase provides comprehensive stamp-based loyalty program functionality within the Buzzebees SDK. It enables users to create stamps, manage their stamp collections, and track detailed progress information including transaction history and campaign participation. The UseCase supports device-based validation and comprehensive loyalty program management.
