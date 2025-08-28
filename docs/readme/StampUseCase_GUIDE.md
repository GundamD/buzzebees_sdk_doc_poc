## StampUseCase Guide

This guide shows how to initialize and use every public method in `StampUseCase`, with suspend and
callback examples where available. The StampUseCase provides comprehensive digital stamp
functionality for loyalty programs, allowing users to create stamp cards, retrieve stamp lists, and
manage stamp profiles with detailed history and campaign information.

### Getting an instance

```kotlin
val stampService = BuzzebeesSDK.instance().stampUseCase
```

---

### createStamp

Creates a new digital stamp card for a user with specified device and issuer information.

- Request (caller-supplied)

| Field Name | Description             | Mandatory | Data Type |
|------------|-------------------------|-----------|-----------|
| imei       | Device IMEI identifier  | M         | String    |
| issuer     | Stamp issuer identifier | M         | String    |
| os         | Operating system        | M         | String    |
| platform   | Platform type           | M         | String    |

- Response (`CreateStampResponse`)
  HTTP status: 200

### CreateStampResponse Entity Fields

| Field Name | Description               | Data Type | JSON Field |
|------------|---------------------------|-----------|------------|
| success    | Creation success status   | Boolean?  | success    |
| balance    | Current stamp balance     | Double?   | balance    |
| cardId     | Generated card identifier | String?   | cardId     |
| userId     | User identifier           | String?   | userId     |
| imei       | Device IMEI               | String?   | imei       |
| appId      | Application identifier    | String?   | app_id     |
| issuer     | Stamp issuer              | String?   | issuer     |
| stampId    | Stamp identifier          | String?   | stampId    |
| sponsorId  | Sponsor identifier        | Int?      | sponsorId  |

- Usage

```kotlin
// Suspend
val result = stampService.createStamp(
    imei = "123456789012345",
    issuer = "coffee_shop_xyz",
    os = "Android",
    platform = "mobile"
)

// Callback
stampService.createStamp("123456789012345", "coffee_shop_xyz", "Android", "mobile") { result ->
    when (result) {
        is StampResult.SuccessCreate -> {
            // Handle successful stamp creation
            val createResponse = result.result

            if (createResponse.success == true) {
                println("Stamp card created successfully!")
                println("Card ID: ${createResponse.cardId}")
                println("Stamp ID: ${createResponse.stampId}")
                println("Balance: ${createResponse.balance}")
            }
        }
        is StampResult.Error -> {
            // Handle error
            println("Error: ${result.error.message}")
        }
    }
}
```

---

### getStampList

Retrieves a list of all stamp cards associated with the current user.

- Request (caller-supplied)

No parameters required.

- Response (`List<Stamp>`)
  HTTP status: 200

### Stamp Entity Fields

| Field Name        | Description               | Data Type | JSON Field        |
|-------------------|---------------------------|-----------|-------------------|
| amount            | Stamp amount/value        | Double?   | amount            |
| agencyId          | Agency identifier         | Int?      | agencyId          |
| cardId            | Card identifier           | String?   | cardId            |
| issuer            | Stamp issuer              | String?   | issuer            |
| stampId           | Stamp identifier          | String?   | stampId           |
| imageUrl          | Stamp image URL           | String?   | imageUrl          |
| owner             | Stamp owner               | String?   | owner             |
| timestamp         | Creation timestamp        | Int?      | timestamp         |
| active            | Whether stamp is active   | Boolean?  | active            |
| name              | Stamp name/title          | String?   | name              |
| description       | Stamp description         | String?   | description       |
| stampImageUrl     | Stamp design image URL    | String?   | stampImageUrl     |
| maxQuantity       | Maximum stamp quantity    | Int?      | maxQuantity       |
| currentQuantity   | Current stamp count       | Int?      | currentQuantity   |
| pricePerStamp     | Price per stamp           | Double?   | pricePerStamp     |
| codeExpireIn      | Code expiration time      | Int?      | codeExpireIn      |
| stampRawScore     | Raw stamp score           | Int?      | stampRawScore     |
| stampScore        | Calculated stamp score    | Int?      | stampScore        |
| stampSpecialScore | Special stamp bonus score | Int?      | stampSpecialScore |

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
                println("${stamp.name} (${stamp.issuer})")
                println("Progress: ${stamp.currentQuantity}/${stamp.maxQuantity}")
                println("Active: ${stamp.active}")
            }
        }
        is StampResult.Error -> {
            // Handle error
            println("Error: ${result.error.message}")
        }
    }
}
```

---

### getStampProfile

Retrieves detailed profile information for a specific stamp card, including campaigns, history, and
related promotions.

- Request (caller-supplied)

| Field Name | Description      | Mandatory | Data Type |
|------------|------------------|-----------|-----------|
| stampId    | Stamp identifier | M         | String    |
| cardId     | Card identifier  | M         | String    |

- Response (`StampProfileResponse`)
  HTTP status: 200

### StampProfileResponse Entity Fields

| Field Name      | Description                | Data Type            | JSON Field      |
|-----------------|----------------------------|----------------------|-----------------|
| id              | Profile identifier         | String?              | id              |
| agencyId        | Agency identifier          | Int?                 | agencyId        |
| cardId          | Card identifier            | String?              | cardId          |
| name            | Stamp name                 | String?              | name            |
| description     | Stamp description          | String?              | description     |
| imageUrl        | Profile image URL          | String?              | imageUrl        |
| backgroundUrl   | Background image URL       | String?              | backgroundUrl   |
| maxQuantity     | Maximum stamp quantity     | Int?                 | maxQuantity     |
| currentQuantity | Current stamp count        | Int?                 | currentQuantity |
| expireDate      | Expiration date timestamp  | Long?                | expireDate      |
| pricePerStamp   | Price per stamp            | Double?              | pricePerStamp   |
| otherPromotions | Other available promotions | Any?                 | otherPromotions |
| otherStamps     | Related stamp cards        | `List<Stamp>?`         | otherStamps     |
| campaigns       | Available campaigns        | `List<StampCampaign>?` | campaigns       |
| history         | Stamp usage history        | `List<StampHistory>?`  | history         |

### StampCampaign Entity Fields

| Field Name | Description        | Data Type | JSON Field |
|------------|--------------------|-----------|------------|
| id         | Campaign ID        | Int?      | id         |
| imgUrl     | Campaign image URL | String?   | img_url    |
| qty        | Required quantity  | Int?      | qty        |

### StampHistory Entity Fields

| Field Name    | Description             | Data Type | JSON Field    |
|---------------|-------------------------|-----------|---------------|
| amount        | Transaction amount      | Double?   | amount        |
| terminalId    | Terminal identifier     | String?   | terminalId    |
| branchId      | Branch identifier       | String?   | branchId      |
| branchName    | Branch name             | String?   | branchName    |
| brandId       | Brand identifier        | String?   | brandId       |
| customerId    | Customer identifier     | String?   | customerId    |
| customerName  | Customer name           | Any?      | customerName  |
| campaignId    | Campaign identifier     | Any?      | campaignId    |
| description   | Transaction description | String?   | description   |
| issuer        | Transaction issuer      | String?   | issuer        |
| issuerName    | Issuer name             | String?   | issuerName    |
| imageUrl      | Transaction image URL   | String?   | imageUrl      |
| merchant      | Merchant identifier     | String?   | merchant      |
| merchantName  | Merchant name           | String?   | merchantName  |
| status        | Transaction status      | Int?      | status        |
| timestamp     | Transaction timestamp   | Int?      | timestamp     |
| type          | Transaction type        | String?   | type          |
| stampCount    | Number of stamps        | Int?      | stampCount    |
| createDate    | Creation date           | Long?     | createDate    |
| name          | Transaction name        | String?   | name          |
| backgroundUrl | Background image URL    | String?   | backgroundUrl |

- Usage

```kotlin
// Suspend
val result = stampService.getStampProfile(
    stampId = "stamp_123",
    cardId = "card_456"
)

// Callback
stampService.getStampProfile("stamp_123", "card_456") { result ->
    when (result) {
        is StampResult.SuccessStampProfile -> {
            // Handle successful stamp profile retrieval
            val profile = result.result

            println("Name: ${profile.name}")
            println("Progress: ${profile.currentQuantity}/${profile.maxQuantity}")
            println("Expires: ${profile.expireDate?.let { java.util.Date(it) }}")

            // Display campaigns
            profile.campaigns?.forEach { campaign ->
                println("Campaign ${campaign.id}: ${campaign.qty} stamps needed")
            }

            // Display history
            profile.history?.forEach { history ->
                println("${history.description} - ${history.stampCount} stamps")
            }
        }
        is StampResult.Error -> {
            // Handle error
            println("Error: ${result.error.message}")
        }
    }
}
```

---

## Summary

The StampUseCase provides digital stamp card management functionality within the Buzzebees SDK. It
offers three methods:

- **createStamp**: Initialize new digital stamp cards for users
- **getStampList**: Retrieve all stamp cards associated with the current user
- **getStampProfile**: Get detailed information about specific stamp cards including campaigns and
  history

Each method supports both suspend and callback patterns with comprehensive error handling for
building loyalty program interfaces.
