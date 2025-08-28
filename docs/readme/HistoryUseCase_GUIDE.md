## HistoryUseCase Guide

This guide shows how to initialize and use every public method in `HistoryUseCase`, with suspend
and callback examples where available. The HistoryUseCase provides comprehensive history management
functionality for retrieving user purchase history and using redeemed campaigns.

### Getting an instance

```kotlin
val historyService = BuzzebeesSDK.instance().historyUseCase
```

---

### getHistory

Retrieves the user's purchase and redemption history with filtering and pagination options.

- Request (caller-supplied)

| Field Name | Description                                    | Mandatory | Data Type |
|------------|------------------------------------------------|-----------|-----------|
| byConfig   | Filter by configuration flag                   | O         | Boolean   |
| config     | Configuration identifier from Backoffice      | M         | String    |
| skip       | Number of records to skip for pagination      | O         | Int?      |
| top        | Maximum number of records to return            | O         | Int?      |
| locale     | User language (1054: Thai, 1033: English)     | O         | Int?      |
| startDate  | Filter start date (ISO format string)         | O         | String?   |
| endDate    | Filter end date (ISO format string)           | O         | String?   |

- Response (`List<Purchase>`)
  HTTP status: 200

### Purchase Entity Fields

| Field Name               | Description                          | Data Type | JSON Field               |
|--------------------------|--------------------------------------|-----------|--------------------------|
| agencyName               | Agency/merchant name                 | String?   | AgencyName               |
| iD                       | Purchase/campaign identifier         | Int?      | ID                       |
| agencyID                 | Agency identifier                    | Int?      | AgencyID                 |
| name                     | Campaign/item name                   | String?   | Name                     |
| detail                   | Campaign/item details                | String?   | Detail                   |
| condition                | Usage conditions and terms           | String?   | Condition                |
| categoryID               | Category identifier                  | Int?      | CategoryID               |
| categoryName             | Category display name                | String?   | CategoryName             |
| startDate                | Campaign start date timestamp        | Long?     | StartDate                |
| expireDate               | Campaign expiration date timestamp   | Long?     | ExpireDate               |
| location                 | Physical location or store address   | String?   | Location                 |
| website                  | Campaign website URL                 | String?   | Website                  |
| discount                 | Discount amount or percentage        | Double?   | Discount                 |
| originalPrice            | Original price before discount       | Double?   | OriginalPrice            |
| pricePerUnit             | Price per unit                       | Double?   | PricePerUnit             |
| pointPerUnit             | Points required per unit             | Double?   | PointPerUnit             |
| quantity                 | Available quantity                   | Double?   | Quantity                 |
| redeemMostPerPerson      | Maximum redemptions per person       | Double?   | RedeemMostPerPerson      |
| peopleLike               | Number of likes                      | Int?      | PeopleLike               |
| peopleDislike            | Number of dislikes                   | Int?      | PeopleDislike            |
| itemCountSold            | Number of items sold                 | Double?   | ItemCountSold            |
| delivered                | Delivery status flag                 | Boolean?  | Delivered                |
| buzz                     | Buzz rating or score                 | Int?      | Buzz                     |
| type                     | Campaign type identifier             | Int?      | Type                     |
| soldOutDate              | Date when sold out                   | Long?     | SoldOutDate              |
| itemNumber               | Unique item number                   | Int?      | ItemNumber               |
| useExpireDate            | Usage expiration date                | Long?     | UseExpireDate            |
| serial                   | Serial number or voucher code        | String?   | Serial                   |
| redeemDate               | Date when redeemed                   | Long?     | RedeemDate               |
| isUsed                   | Usage status flag                    | Boolean?  | IsUsed                   |
| isWinner                 | Winner status for contests           | Boolean?  | IsWinner                 |
| usedDate                 | Date when used                       | Long?     | UsedDate                 |
| winningDate              | Date of winning (for contests)       | Long?     | WinningDate              |
| shippingDate             | Date when shipped                    | Long?     | ShippingDate             |
| isShipped                | Shipping status flag                 | Boolean?  | IsShipped                |
| minutesValidAfterUsed    | Validity minutes after use           | Int?      | MinutesValidAfterUsed    |
| hasWinner                | Contest has winner flag              | Boolean?  | HasWinner                |
| voucherExpireDate        | Voucher expiration date              | Long?     | VoucherExpireDate        |
| currentDate              | Current server date                  | Long?     | CurrentDate              |
| barcode                  | Barcode for scanning                 | String?   | Barcode                  |
| parcelNo                 | Parcel tracking number               | String?   | ParcelNo                 |
| expireIn                 | Minutes until expiration             | Int?      | ExpireIn                 |
| info1                    | Additional information field 1       | String?   | Info1                    |
| info2                    | Additional information field 2       | String?   | Info2                    |
| info3                    | Additional information field 3       | String?   | Info3                    |
| info4                    | Additional information field 4       | String?   | Info4                    |
| info5                    | Additional information field 5       | String?   | Info5                    |
| interfaceDisplay         | UI display configuration             | String?   | InterfaceDisplay         |
| pointType                | Points type identifier               | String?   | PointType                |
| contactNumber            | Contact phone number                 | String?   | ContactNumber            |
| isNotAutoUse             | Prevent automatic usage flag         | Boolean?  | IsNotAutoUse             |
| parentCategoryID         | Parent category identifier           | Int?      | ParentCategoryID         |
| privilegeMessage         | Privilege message in default lang    | String?   | PrivilegeMessage         |
| privilegeMessageEN       | Privilege message in English         | String?   | PrivilegeMessageEN       |
| privilegeMessageFormat   | Privilege message format type        | String?   | PrivilegeMessageFormat   |
| redeemKey                | Unique redemption key                | String?   | RedeemKey                |
| isConditionPass          | Condition validation status          | Boolean?  | IsConditionPass          |
| isExpired                | Expiration status flag               | Boolean?  | IsExpired                |
| fullImageUrl             | Full resolution image URL            | String?   | FullImageUrl             |
| arrangedDate             | Arranged delivery date               | String?   | ArrangedDate             |

- Usage

```kotlin
// Create form
val historyForm = HistoryForm(
    byConfig = true,
    config = "my_app_config",
    skip = 0,
    top = 20,
    locale = 1054,
    startDate = "2024-01-01T00:00:00Z",
    endDate = "2024-12-31T23:59:59Z"
)

// Suspend
val result = historyService.getHistory(historyForm)

// Callback
historyService.getHistory(historyForm) { result ->
    when (result) {
        is HistoryResult.SuccessList -> {
            // Handle successful history retrieval
            val purchaseHistory = result.result

            purchaseHistory.forEach { purchase ->
                val campaignName = purchase.name
                val agencyName = purchase.agencyName
                val redeemDate = purchase.redeemDate
                val isUsed = purchase.isUsed

                println("Campaign: $campaignName")
                println("Agency: $agencyName")
            }
        }
        is HistoryResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### getHistoryUse

Uses a redeemed campaign or voucher by providing the redemption key.

- Request (caller-supplied)

| Field Name | Description                     | Mandatory | Data Type |
|------------|---------------------------------|-----------|-----------|
| redeemKey  | Unique redemption key from history | M       | String    |

- Response (`UseCampaignResponse`)
  HTTP status: 200

### UseCampaignResponse Entity Fields

| Field Name               | Description                          | Data Type | JSON Field               |
|--------------------------|--------------------------------------|-----------|--------------------------|
| campaignId               | Campaign identifier                  | Int?      | CampaignId               |
| itemNumber               | Unique item number                   | Int?      | ItemNumber               |
| serial                   | Serial number or voucher code        | String?   | Serial                   |
| privilegeMessage         | Privilege message after usage        | String?   | PrivilegeMessage         |
| usedDate                 | Date when used (timestamp)           | Long?     | UsedDate                 |
| currentDate              | Current server date (timestamp)      | Long?     | CurrentDate              |
| privilegeMessageFormat   | Message format type                  | String?   | PrivilegeMessageFormat   |
| redeemCount              | Total redemption count               | Int?      | RedeemCount              |
| useCount                 | Total usage count                    | Int?      | UseCount                 |
| expireIn                 | Minutes until expiration             | Int?      | ExpireIn                 |

- Usage

```kotlin
// Suspend
val result = historyService.getHistoryUse("redeem_key_12345")

// Callback
historyService.getHistoryUse("redeem_key_from_purchase_history") { result ->
    when (result) {
        is HistoryResult.SuccessUse -> {
            // Handle successful campaign usage
            val useResponse = result.result

            val campaignId = useResponse.campaignId
            val privilegeMessage = useResponse.privilegeMessage
            val usedDate = useResponse.usedDate

            println("Campaign $campaignId used successfully!")
            println("Privilege: $privilegeMessage")
        }
        is HistoryResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

## Summary

The HistoryUseCase provides comprehensive purchase and redemption history management functionality within the Buzzebees SDK. It offers detailed filtering capabilities, pagination support, and campaign usage functionality for tracking purchase history and voucher usage.
