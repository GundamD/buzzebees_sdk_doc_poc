## CampaignUseCase Guide

This guide shows how to initialize and use every public method in `CampaignUseCase`, with suspend
and callback examples where available. The CampaignUseCase provides comprehensive campaign
management functionality including browsing, favorites, redemption, and review systems for
promotional campaigns and offers.

### Getting an instance

```kotlin
val campaignService = BuzzebeesSDK.instance().campaignUseCase
```

---

### getCampaignList

Retrieves a list of campaigns based on various filtering criteria and configuration parameters.

- Request (caller-supplied)

| Field Name | Description               | Mandatory | Data Type        |
|------------|---------------------------|-----------|------------------|
| form       | Campaign list filter form | M         | CampaignListForm |

#### CampaignListForm Fields

| Field Name       | Description                | Mandatory | Data Type           |
|------------------|----------------------------|-----------|---------------------|
| config           | Configuration identifier   | M         | String              |
| cat              | Category filter            | O         | String?             |
| byConfig         | Filter by configuration    | O         | Boolean             |
| skip             | Number of items to skip    | O         | Int? (default: 0)   |
| top              | Number of items to return  | O         | Int? (default: 25)  |
| deviceLocale     | Device locale identifier   | O         | Int?                |
| locale           | Locale identifier          | O         | Int?                |
| keyword          | Search keyword             | O         | String?             |
| startDate        | Filter from start date     | O         | String?             |
| sponsorId        | Sponsor identifier         | O         | String?             |
| maxPoints        | Maximum points filter      | O         | String?             |
| minPoints        | Minimum points filter      | O         | String?             |
| sortBy           | Sort criteria              | O         | String?             |
| center           | Geographic center          | O         | String?             |
| hashTags         | Hashtag filters            | O         | String?             |
| locationAgencyId | Location agency identifier | O         | String?             |
| options          | Additional options         | O         | Map<String, String> |

- Response (`List<Campaign>`) [source](../buzzebees_sdk/src/main/java/com/buzzebees/sdk/entity/campaign/Campaign.kt)  
  HTTP status: 200

### Campaign Entity Fields

| Field Name            | Description                     | Data Type    | JSON Field            |
|-----------------------|---------------------------------|--------------|-----------------------|
| iD                    | Unique campaign identifier      | Int?         | ID                    |
| referenceCode         | Campaign reference code         | String?      | ReferenceCode         |
| name                  | Campaign display name           | String?      | Name                  |
| agencyId              | Agency identifier               | Int?         | AgencyId              |
| agencyName            | Agency display name             | String?      | AgencyName            |
| startDate             | Campaign start timestamp        | Long?        | StartDate             |
| isConditionPass       | User meets campaign conditions  | Boolean?     | IsConditionPass       |
| dayProceed            | Days since campaign started     | Int?         | DayProceed            |
| dayRemain             | Days remaining until expiration | Int?         | DayRemain             |
| qty                   | Available quantity              | Double?      | Qty                   |
| redeemMostPerPerson   | Max redemptions per person      | Double?      | RedeemMostPerPerson   |
| redeemCount           | Current redemption count        | Int?         | RedeemCount           |
| currentDate           | Current server timestamp        | Long?        | CurrentDate           |
| nextRedeemDate        | Next available redemption date  | Long?        | NextRedeemDate        |
| pointPerUnit          | Points required per unit        | Double?      | PointPerUnit          |
| pricePerUnit          | Price per unit                  | Double?      | PricePerUnit          |
| discount              | Discount amount/percentage      | Double?      | Discount              |
| type                  | Campaign type identifier        | Int?         | Type                  |
| caption               | Campaign caption/subtitle       | String?      | Caption               |
| isSponsor             | Is sponsored campaign           | Boolean?     | IsSponsor             |
| fullImageUrl          | Campaign image URL              | String?      | FullImageUrl          |
| expireDate            | Campaign expiration timestamp   | Long?        | ExpireDate            |
| extra                 | Extra campaign information      | String?      | Extra                 |
| originalPrice         | Original price before discount  | Double?      | OriginalPrice         |
| pointType             | Type of points (coins, stamps)  | String?      | PointType             |
| categoryId            | Category identifier             | Int?         | CategoryId            |
| categoryName          | Category display name           | String?      | CategoryName          |
| parentCategoryId      | Parent category identifier      | Int?         | ParentCategoryId      |
| masterCampaignId      | Master campaign identifier      | Int?         | MasterCampaignId      |
| like                  | Like count                      | Int?         | Like                  |
| buzz                  | Buzz/engagement score           | Int?         | Buzz                  |
| website               | Campaign website URL            | String?      | Website               |
| itemCountSold         | Total items sold                | Double?      | ItemCountSold         |
| subCampaigns          | Sub-campaign information        | SubCampaign? | SubCampaigns          |
| customFacebookMessage | Custom Facebook sharing message | String?      | CustomFacebookMessage |

### SubCampaign Entity Fields

| Field Name          | Description                | Data Type | JSON Field          |
|---------------------|----------------------------|-----------|---------------------|
| CampaignId          | Campaign identifier        | Int?      | CampaignId          |
| Type                | Sub-campaign type          | String?   | Type                |
| Size                | Campaign size              | String?   | Size                |
| Quantity            | Available quantity         | Double?   | Quantity            |
| RedeemMostPerPerson | Max redemptions per person | Int?      | RedeemMostPerPerson |
| ItemCountSold       | Items sold count           | Int?      | ItemCountSold       |
| imageUrl            | Sub-campaign image URL     | String?   | imageUrl            |

- Usage

```kotlin
// Create campaign list form
val campaignForm = CampaignListForm(
    config = "rewards",
    cat = "food",
    byConfig = true,
    skip = 0,
    top = 20,
    deviceLocale = 1054, // Thai locale
    keyword = "coffee",
    sortBy = "startDate",
    options = mapOf("includeExpired" to "false")
)

// Suspend
val result = campaignService.getCampaignList(campaignForm)

// Callback
campaignService.getCampaignList(campaignForm) { result ->
    when (result) {
        is CampaignResult.SuccessCampaignList -> {
            // Handle successful campaign list retrieval
            val campaigns = result.result
            campaigns.forEach { campaign ->
                val name = campaign.name
                val points = campaign.pointPerUnit
                val price = campaign.pricePerUnit
                val expireDate = campaign.expireDate
                val redeemCount = campaign.redeemCount
                val imageUrl = campaign.fullImageUrl

                println("Campaign: $name - Points: $points, Price: $price")
                println("Remaining: ${campaign.qty}, Redeemed: $redeemCount")
            }
        }
        is CampaignResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

#### Error Handling

| Error Code | Error ID | Scenario           | User Message                             | Recommended Action                               |
|------------|----------|--------------------|------------------------------------------|--------------------------------------------------|
| 403        | 403      | No permission      | "No permission to view this campaign"    | Hide campaign or show upgrade membership message |
| 404        | -        | Campaign not found | "Campaign not found"                     | Show empty state or suggest other campaigns      |
| 500        | -        | Server error       | "Something went wrong. Please try again" | Show "Retry" button                              |

#### Implementation Example

```kotlin
campaignService.getCampaignList(campaignForm) { result ->
    when (result) {
        is CampaignResult.SuccessCampaignList -> {
            displayCampaigns(result.result)
        }
        is CampaignResult.Error -> {
            val action = when (result.error.code) {
                "403" -> ErrorAction.ShowNoPermissionMessage()
                "404" -> ErrorAction.ShowEmptyState()
                "500" -> ErrorAction.ShowRetryOption()
                else -> ErrorAction.ShowGenericError(result.error.message)
            }
            handleErrorAction(action)
        }
    }
}
```

---

### getCampaignDetail

Retrieves detailed information for a specific campaign.

- Request (caller-supplied)

| Field Name   | Description         | Mandatory | Data Type           |
|--------------|---------------------|-----------|---------------------|
| id           | Campaign identifier | M         | String              |
| deviceLocale | Device locale       | O         | Int?                |
| options      | Additional options  | O         | Map<String, String> |

- Response (`CampaignDetails`) [source](../buzzebees_sdk/src/main/java/com/buzzebees/sdk/entity/campaign/CampaignDetails.kt)  
  HTTP status: 200

### CampaignDetails Entity Fields

| Field Name              | Description                      | Data Type          | JSON Field              |
|-------------------------|----------------------------------|--------------------|-------------------------|
| agencyName              | Agency display name              | String?            | AgencyName              |
| id                      | Campaign identifier              | Int?               | ID                      |
| agencyID                | Agency identifier                | Int?               | AgencyID                |
| name                    | Campaign display name            | String?            | Name                    |
| detail                  | Detailed campaign description    | String?            | Detail                  |
| condition               | Campaign terms and conditions    | String?            | Condition               |
| conditionAlert          | Condition alert message          | String?            | ConditionAlert          |
| categoryID              | Category identifier              | Int?               | CategoryID              |
| categoryName            | Category display name            | String?            | CategoryName            |
| startDate               | Campaign start timestamp         | Long?              | StartDate               |
| currentDate             | Current server timestamp         | Long?              | CurrentDate             |
| expireDate              | Campaign expiration timestamp    | Long?              | ExpireDate              |
| location                | Campaign location information    | String?            | Location                |
| website                 | Campaign website URL             | String?            | Website                 |
| discount                | Discount amount/percentage       | Double?            | Discount                |
| originalPrice           | Original price before discount   | Double?            | OriginalPrice           |
| pricePerUnit            | Price per unit                   | Double?            | PricePerUnit            |
| pointPerUnit            | Points required per unit         | Double?            | PointPerUnit            |
| quantity                | Available quantity               | Double?            | Quantity                |
| redeemMostPerPerson     | Max redemptions per person       | Double?            | RedeemMostPerPerson     |
| peopleLike              | Number of people who liked       | Int?               | PeopleLike              |
| peopleDislike           | Number of people who disliked    | Int?               | PeopleDislike           |
| itemCountSold           | Total items sold                 | Double?            | ItemCountSold           |
| delivered               | Campaign delivery status         | Boolean?           | Delivered               |
| buzz                    | Buzz/engagement score            | Int?               | Buzz                    |
| type                    | Campaign type identifier         | Int?               | Type                    |
| isSponsor               | Is sponsored campaign            | Boolean?           | IsSponsor               |
| dayRemain               | Days remaining until expiration  | Int?               | DayRemain               |
| dayProceed              | Days since campaign started      | Int?               | DayProceed              |
| soldOutDate             | Date when campaign sold out      | Long?              | SoldOutDate             |
| caption                 | Campaign caption/subtitle        | String?            | Caption                 |
| voucherExpireDate       | Voucher expiration timestamp     | Long?              | VoucherExpireDate       |
| userLevel               | Required user level              | Long?              | UserLevel               |
| redeemCount             | Current redemption count         | Int?               | RedeemCount             |
| useCount                | Usage count                      | Int?               | UseCount                |
| nextRedeemDate          | Next available redemption date   | Long?              | NextRedeemDate          |
| isLike                  | User liked this campaign         | Boolean?           | IsLike                  |
| minutesValidAfterUsed   | Minutes valid after use          | Int?               | MinutesValidAfterUsed   |
| barcode                 | Campaign barcode                 | String?            | Barcode                 |
| customInput             | Custom input field               | String?            | CustomInput             |
| customCaption           | Custom caption text              | String?            | CustomCaption           |
| customFacebookMessage   | Custom Facebook sharing message  | String?            | CustomFacebookMessage   |
| interfaceDisplay        | Interface display configuration  | String?            | InterfaceDisplay        |
| pointType               | Type of points (coins, stamps)   | String?            | PointType               |
| defaultPrivilegeMessage | Default privilege message        | String?            | DefaultPrivilegeMessage |
| isNotAutoUse            | Not auto-use flag                | Boolean?           | IsNotAutoUse            |
| pictures                | Campaign image gallery           | List<Picture>?     | Pictures                |
| isConditionPass         | User meets campaign conditions   | Boolean?           | IsConditionPass         |
| conditionAlertId        | Condition alert identifier       | Int?               | ConditionAlertId        |
| qty                     | Available quantity               | Double?            | Qty                     |
| fullImageUrl            | Full campaign image URL          | String?            | FullImageUrl            |
| subCampaignStyles       | Sub-campaign style configuration | SubCampaignStyle?  | SubCampaignStyles       |
| subCampaigns            | List of sub-campaigns            | List<SubCampaign>? | SubCampaigns            |
| related                 | Related campaigns                | List<Campaign>?    | Related                 |
| isFavourite             | User marked as favorite          | Boolean?           | IsFavourite             |

### Picture Entity Fields

| Field Name   | Description            | Data Type | JSON Field   |
|--------------|------------------------|-----------|--------------|
| iD           | Picture identifier     | Int?      | ID           |
| campaignID   | Campaign identifier    | Int?      | CampaignID   |
| type         | Picture type           | String?   | Type         |
| sequence     | Picture sequence order | Int?      | Sequence     |
| caption      | Picture caption        | String?   | Caption      |
| imageUrl     | Picture URL            | String?   | ImageUrl     |
| fullImageUrl | Full picture URL       | String?   | FullImageUrl |

### SubCampaignStyle Entity Fields

| Field Name       | Description                 | Data Type         | JSON Field       |
|------------------|-----------------------------|-------------------|------------------|
| styles           | List of style options       | ArrayList<Style>? | styles           |
| masterCampaignId | Master campaign identifier  | String?           | masterCampaignId |
| isHasStyle       | Has style options available | Boolean?          | hasStyle         |

### Style Entity Fields

| Field Name    | Description                | Data Type           | JSON Field    |
|---------------|----------------------------|---------------------|---------------|
| value         | Style value                | String?             | value         |
| points        | Points required            | Double?             | points        |
| subitems      | List of sub-items          | ArrayList<SubItem>? | subitems      |
| price         | Style price                | Double?             | price         |
| campaignId    | Campaign identifier        | String?             | campaignId    |
| name          | Style name                 | String?             | name          |
| nameEn        | Style name in English      | String?             | name_en       |
| quantity      | Available quantity         | Double?             | quantity      |
| itemCountSold | Items sold count           | Int?                | itemCountSold |
| originalPrice | Original price             | Double?             | originalPrice |
| discount      | Discount amount            | Double?             | discount      |
| sequence      | Style sequence order       | Int?                | sequence      |
| partialPoints | Partial points information | Any?                | partialPoints |

### SubItem Entity Fields

| Field Name    | Description           | Data Type | JSON Field    |
|---------------|-----------------------|-----------|---------------|
| value         | Sub-item value        | String?   | value         |
| type          | Sub-item type         | String?   | type          |
| points        | Points required       | Double?   | points        |
| price         | Sub-item price        | Double?   | price         |
| itemCountSold | Items sold count      | Int?      | itemCountSold |
| campaignId    | Campaign identifier   | String?   | campaignId    |
| name          | Sub-item name         | String?   | name          |
| quantity      | Available quantity    | Double?   | quantity      |
| originalPrice | Original price        | Double?   | originalPrice |
| selected      | Selection status      | Boolean?  | selected      |
| discount      | Discount amount       | Double?   | discount      |
| sequence      | Sub-item sequence     | Int?      | sequence      |
| partialPoints | Partial points info   | Any?      | partialPoints |
| nameEn        | Sub-item name English | String?   | name_en       |

- Usage

```kotlin
// Suspend
val result = campaignService.getCampaignDetail(
    id = "12345",
    deviceLocale = 1054,
    options = mapOf("includeReviews" to "true")
)

// Callback
campaignService.getCampaignDetail("12345", 1054) { result ->
    when (result) {
        is CampaignResult.SuccessCampaignDetail -> {
            // Handle successful campaign detail retrieval
            val campaignDetail = result.result
            // Access detailed campaign information
            val description = campaignDetail.detail
            val terms = campaignDetail.condition
            val images = campaignDetail.pictures
            val styles = campaignDetail.subCampaignStyles
        }
        is CampaignResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### getFavoriteList

Retrieves a list of campaigns marked as favorites by the user.

- Request (caller-supplied)

| Field Name | Description               | Mandatory | Data Type           |
|------------|---------------------------|-----------|---------------------|
| skip       | Number of items to skip   | O         | Int?                |
| top        | Number of items to return | O         | Int?                |
| locale     | Locale identifier         | O         | Int?                |
| options    | Additional options        | O         | Map<String, String> |

- Response (`List<Campaign>`) [source](../buzzebees_sdk/src/main/java/com/buzzebees/sdk/entity/campaign/Campaign.kt)  
  HTTP status: 200

> **Campaign Fields Reference**
>
> Same as getCampaignList response - see the complete Campaign entity fields table above.

- Usage

```kotlin
// Suspend
val result = campaignService.getFavoriteList(
    skip = 0,
    top = 10,
    locale = 1054
)

// Callback
campaignService.getFavoriteList(0, 10, 1054) { result ->
    when (result) {
        is CampaignResult.SuccessFavoriteList -> {
            // Handle successful favorite list retrieval
            val favoriteCampaigns = result.result
            favoriteCampaigns.forEach { campaign ->
                val name = campaign.name
                val agencyName = campaign.agencyName
                println("Favorite: $name from $agencyName")
            }
        }
        is CampaignResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### getFavoriteCampaign

Adds a campaign to the user's favorites list.

- Request (caller-supplied)

| Field Name | Description         | Mandatory | Data Type |
|------------|---------------------|-----------|-----------|
| id         | Campaign identifier | M         | String    |

- Response (`FavoriteResponse`) [source](../buzzebees_sdk/src/main/java/com/buzzebees/sdk/entity/campaign/FavoriteResponse.kt)  
  HTTP status: 200

### FavoriteResponse Entity Fields

| Field Name      | Description                    | Data Type | JSON Field      |
|-----------------|--------------------------------|-----------|-----------------|
| peopleFavourite | Number of people who favorited | Int?      | PeopleFavourite |
| favourite       | Favorite status indicator      | Int?      | Favourite       |

- Usage

```kotlin
// Suspend
val result = campaignService.getFavoriteCampaign("12345")

// Callback
campaignService.getFavoriteCampaign("12345") { result ->
    when (result) {
        is CampaignResult.SuccessFavoriteCampaign -> {
            // Handle successful favorite addition
            val response = result.result
            val peopleCount = response.peopleFavourite
            val favoriteStatus = response.favourite
            // Campaign successfully added to favorites
        }
        is CampaignResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### unFavoriteCampaign

Removes a campaign from the user's favorites list.

- Request (caller-supplied)

| Field Name | Description         | Mandatory | Data Type |
|------------|---------------------|-----------|-----------|
| id         | Campaign identifier | M         | String    |

- Response (`FavoriteResponse`) [source](../buzzebees_sdk/src/main/java/com/buzzebees/sdk/entity/campaign/FavoriteResponse.kt)  
  HTTP status: 200

### FavoriteResponse Entity Fields

| Field Name      | Description                    | Data Type | JSON Field      |
|-----------------|--------------------------------|-----------|-----------------|
| peopleFavourite | Number of people who favorited | Int?      | PeopleFavourite |
| favourite       | Favorite status indicator      | Int?      | Favourite       |

- Usage

```kotlin
// Suspend
val result = campaignService.unFavoriteCampaign("12345")

// Callback
campaignService.unFavoriteCampaign("12345") { result ->
    when (result) {
        is CampaignResult.SuccessFavoriteCampaign -> {
            // Handle successful favorite removal
            val response = result.result
            val peopleCount = response.peopleFavourite
            val favoriteStatus = response.favourite
            // Campaign successfully removed from favorites
        }
        is CampaignResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### redeem

Redeems a campaign offer using user points or other redemption methods.

- Request (caller-supplied)

| Field Name | Description         | Mandatory | Data Type           |
|------------|---------------------|-----------|---------------------|
| id         | Campaign identifier | M         | String              |
| options    | Additional options  | O         | Map<String, String> |

- Response (`RedeemResponse`) [source](../buzzebees_sdk/src/main/java/com/buzzebees/sdk/entity/campaign/RedeemResponse.kt)  
  HTTP status: 200

### RedeemResponse Entity Fields

| Field Name             | Description                    | Data Type             | JSON Field             |
|------------------------|--------------------------------|-----------------------|------------------------|
| campaignId             | Campaign identifier            | Int?                  | CampaignId             |
| itemNumber             | Item number                    | Int?                  | ItemNumber             |
| serial                 | Serial number                  | String?               | Serial                 |
| agencyId               | Agency identifier              | Int?                  | AgencyId               |
| agencyName             | Agency display name            | String?               | AgencyName             |
| catId                  | Category identifier            | Int?                  | CatId                  |
| name                   | Redemption name                | String?               | Name                   |
| nextRedeemDate         | Next available redemption date | Long?                 | NextRedeemDate         |
| currentDate            | Current server timestamp       | Long?                 | CurrentDate            |
| redeemCount            | Current redemption count       | Int?                  | RedeemCount            |
| useCount               | Usage count                    | Int?                  | UseCount               |
| qty                    | Available quantity             | Double?               | Qty                    |
| isConditionPass        | User meets conditions          | Boolean?              | IsConditionPass        |
| conditionAlert         | Condition alert message        | String?               | ConditionAlert         |
| isNotAutoUse           | Not auto-use flag              | Boolean?              | IsNotAutoUse           |
| pointType              | Type of points used            | String?               | PointType              |
| interfaceDisplay       | Interface display config       | String?               | InterfaceDisplay       |
| redeemKey              | Redemption key                 | String?               | RedeemKey              |
| pricePerUnit           | Price per unit                 | Double?               | PricePerUnit           |
| redeemDate             | Redemption timestamp           | Long?                 | RedeemDate             |
| campaignName           | Campaign display name          | String?               | CampaignName           |
| pointPerUnit           | Points per unit                | Double?               | PointPerUnit           |
| expireIn               | Expiration period              | Long?                 | ExpireIn               |
| privilegeMessage       | Privilege message              | String?               | PrivilegeMessage       |
| privilegeMessageEN     | Privilege message in English   | String?               | PrivilegeMessageEN     |
| privilegeMessageFormat | Privilege message format       | String?               | PrivilegeMessageFormat |
| buzzebees              | Buzzebees trace information    | Buzzebees?            | buzzebees              |
| result                 | Nested redemption results      | List<RedeemResponse>? | Result                 |

### Buzzebees Entity Fields

| Field Name | Description          | Data Type    | JSON Field |
|------------|----------------------|--------------|------------|
| traces     | List of trace events | List<Trace>? | traces     |

### Trace Entity Fields

| Field Name         | Description           | Data Type | JSON Field         |
|--------------------|-----------------------|-----------|--------------------|
| id                 | Trace identifier      | String?   | Id                 |
| current            | Current value         | Int?      | Current            |
| change             | Change amount         | Int?      | Change             |
| points             | Points involved       | Int?      | Points             |
| objectId           | Object identifier     | String?   | ObjectId           |
| valueResetReason   | Value reset reason    | String?   | ValueResetReason   |
| userDescription    | User description      | String?   | UserDescription    |
| userLevel          | User level            | Long?     | UserLevel          |
| customInfo         | Custom information    | String?   | CustomInfo         |
| lineMessageSetting | LINE message settings | String?   | LineMessageSetting |

- Usage

```kotlin
// Suspend
val result = campaignService.redeem(
    id = "12345",
    options = mapOf("useWallet" to "true")
)

// Callback
campaignService.redeem("12345", mapOf("deliveryAddress" to "123 Main St")) { result ->
    when (result) {
        is CampaignResult.SuccessRedeem -> {
            // Handle successful redemption
            val redeemResponse = result.result
            val redeemKey = redeemResponse.redeemKey
            val serial = redeemResponse.serial
            val expireIn = redeemResponse.expireIn
            val pointsUsed = redeemResponse.pointPerUnit
            val privilegeMessage = redeemResponse.privilegeMessage
            val traces = redeemResponse.buzzebees?.traces

            println("Redemption successful!")
            println("Key: $redeemKey")
            println("Serial: $serial")
            println("Points used: $pointsUsed")
            println("Expires in: $expireIn")
            println("Message: $privilegeMessage")
        }
        is CampaignResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message

            when (errorCode) {
                "INSUFFICIENT_POINTS" -> {
                    // Handle insufficient points
                }
                "CAMPAIGN_EXPIRED" -> {
                    // Handle expired campaign
                }
                "LIMIT_EXCEEDED" -> {
                    // Handle redemption limit exceeded
                }
            }
        }
    }
}
```

#### Error Handling

| Error Code | Error ID | Scenario                | User Message                                    | Recommended Action                                    |
|------------|----------|-------------------------|-------------------------------------------------|-------------------------------------------------------|
| 409        | 1        | Campaign sold out       | "This campaign is sold out"                     | Suggest similar campaigns                             |
| 409        | 2        | Max redemptions reached | "You have reached the maximum redemption limit" | Show redemption history, suggest other campaigns      |
| 409        | 3        | Already redeemed        | "You have already redeemed this campaign"       | Show "Already Redeemed" status with details           |
| 409        | 1406     | Campaign unavailable    | "This campaign is not available"                | Hide campaign or show "Not Available" status          |
| 409        | 1409     | Campaign expired        | "This campaign has expired"                     | Show expiration date, suggest new campaigns           |
| 409        | 1410     | Campaign not started    | "This campaign has not started yet"             | Show campaign start date                              |
| 409        | 1416     | Must read conditions    | "Please read the terms and conditions first"    | Open dialog showing terms with "Accept" button        |
| 409        | 1417     | Not authorized          | "You are not authorized to redeem this"         | Show redemption conditions (e.g., must be VIP member) |
| 409        | 4025     | Payment error           | "Payment failed"                                | Suggest checking payment method                       |

#### Implementation Example

```kotlin
campaignService.redeem(campaignId, options) { result ->
    when (result) {
        is CampaignResult.SuccessRedeem -> {
            showRedeemSuccess(result.result)
        }
        is CampaignResult.Error -> {
            val action = when {
                result.error.code == "409" && result.error.id == "1" ->
                    ErrorAction.ShowSoldOutMessage()
                result.error.code == "409" && result.error.id == "2" ->
                    ErrorAction.ShowMaxRedemptionReached()
                result.error.code == "409" && result.error.id == "3" ->
                    ErrorAction.ShowAlreadyRedeemed()
                result.error.code == "409" && result.error.id == "1406" ->
                    ErrorAction.ShowCampaignUnavailable()
                result.error.code == "409" && result.error.id == "1409" ->
                    ErrorAction.ShowCampaignExpired()
                result.error.code == "409" && result.error.id == "1410" ->
                    ErrorAction.ShowCampaignNotStarted()
                result.error.code == "409" && result.error.id == "1416" ->
                    ErrorAction.ShowTermsAndConditionsDialog()
                result.error.code == "409" && result.error.id == "1417" ->
                    ErrorAction.ShowNotAuthorized()
                result.error.code == "409" && result.error.id == "4025" ->
                    ErrorAction.ShowPaymentError()
                else ->
                    ErrorAction.ShowGenericError(result.error.message)
            }
            handleErrorAction(action)
        }
    }
}
```

---

### redeemDrawWithQty

Redeems a campaign with specified quantity, typically used for lottery or draw-based campaigns.

- Request (caller-supplied)

| Field Name | Description         | Mandatory | Data Type           |
|------------|---------------------|-----------|---------------------|
| id         | Campaign identifier | M         | String              |
| quantity   | Redemption quantity | M         | String              |
| options    | Additional options  | O         | Map<String, String> |

- Response (`RedeemResponse`) [source](../buzzebees_sdk/src/main/java/com/buzzebees/sdk/entity/campaign/RedeemResponse.kt)  
  HTTP status: 200

> **RedeemResponse Fields Reference**
>
> Same as redeem response - see the complete RedeemResponse entity fields table above, including all
> nested entities (Buzzebees and Trace).

- Usage

```kotlin
// Suspend
val result = campaignService.redeemDrawWithQty(
    id = "draw-123",
    quantity = "5",
    options = mapOf("lotteryType" to "instant")
)

// Callback
campaignService.redeemDrawWithQty("draw-123", mapOf(), "3") { result ->
    when (result) {
        is CampaignResult.SuccessRedeem -> {
            // Handle successful draw redemption
            val redeemResponse = result.result
            val results = redeemResponse.result
            val traces = redeemResponse.buzzebees?.traces
            val redeemKey = redeemResponse.redeemKey

            println("Draw completed!")
            println("Redemption key: $redeemKey")
            println("Results count: ${results?.size}")
            traces?.forEach { trace ->
                println("Trace: ${trace.id}, Points: ${trace.points}")
            }
        }
        is CampaignResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

**Note**: In the callback version, the parameter order is `id`, `options`, `quantity` (different
from suspend version).

```kotlin
// Callback with correct parameter order
campaignService.redeemDrawWithQty("draw-123", mapOf("lotteryType" to "instant"), "3") { result ->
    // Handle result same as above
}
```

---

### reviewList

Retrieves reviews and ratings for a specific campaign with pagination support.

- Request (caller-supplied)

| Field Name | Description                 | Mandatory | Data Type |
|------------|-----------------------------|-----------|-----------|
| campaignId | Campaign identifier         | M         | String    |
| lastRowKey | Last row key for pagination | O         | String?   |

- Response (`List<Review>`) [source](../buzzebees_sdk/src/main/java/com/buzzebees/sdk/entity/campaign/Review.kt)  
  HTTP status: 200

### Review Entity Fields

| Field Name   | Description               | Data Type | JSON Field   |
|--------------|---------------------------|-----------|--------------|
| userId       | User identifier           | String?   | UserId       |
| name         | User display name         | String?   | Name         |
| message      | Review message/comment    | String?   | Message      |
| imageUrl     | Review image URL          | String?   | ImageUrl     |
| width        | Image width               | Int?      | Width        |
| height       | Image height              | Int?      | Height       |
| type         | Review type               | String?   | Type         |
| agencyId     | Agency identifier         | Int?      | AgencyId     |
| agencyName   | Agency display name       | String?   | AgencyName   |
| likes        | Like count                | Int?      | Likes        |
| isLiked      | User liked this review    | Boolean?  | IsLiked      |
| commentCount | Comment count             | Int?      | CommentCount |
| subject      | Review subject            | String?   | Subject      |
| hiddenCount  | Hidden comment count      | Int?      | HiddenCount  |
| createdTime  | Review creation timestamp | Int?      | CreatedTime  |
| buzzKey      | Buzz key                  | String?   | BuzzKey      |
| appId        | Application identifier    | String?   | AppId        |
| os           | Operating system          | String?   | Os           |
| campaignId   | Campaign identifier       | Int?      | CampaignId   |
| partitionKey | Database partition key    | String?   | PartitionKey |
| rowKey       | Database row key          | String?   | RowKey       |

- Usage

```kotlin
// Suspend - First page
val result = campaignService.reviewList(
    campaignId = "12345",
    lastRowKey = null
)

// Callback - With pagination
campaignService.reviewList("12345", "last-row-key-123") { result ->
    when (result) {
        is CampaignResult.SuccessReviewList -> {
            // Handle successful review list retrieval
            val reviews = result.result
            reviews.forEach { review ->
                val userName = review.name
                val message = review.message
                val likes = review.likes
                val createdTime = review.createdTime
                val imageUrl = review.imageUrl
                val isLiked = review.isLiked
                val commentCount = review.commentCount

                println("Review by $userName: $message")
                println("Likes: $likes, Comments: $commentCount")
                println("Created: $createdTime, Liked by user: $isLiked")
            }
        }
        is CampaignResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### postReview / postReviewSuspend

Posts a review for a campaign with optional image attachment.

- Request (caller-supplied)

| Field Name | Description         | Mandatory | Data Type |
|------------|---------------------|-----------|-----------|
| campaignId | Campaign identifier | M         | String    |
| message    | Review message      | M         | String    |
| image      | Optional image file | O         | File?     |

- Response (`Review`) [source](../buzzebees_sdk/src/main/java/com/buzzebees/sdk/entity/campaign/Review.kt)  
  HTTP status: 200

### Review Entity Fields

| Field Name   | Description               | Data Type | JSON Field   |
|--------------|---------------------------|-----------|--------------|
| userId       | User identifier           | String?   | UserId       |
| name         | User display name         | String?   | Name         |
| message      | Review message/comment    | String?   | Message      |
| imageUrl     | Review image URL          | String?   | ImageUrl     |
| width        | Image width               | Int?      | Width        |
| height       | Image height              | Int?      | Height       |
| type         | Review type               | String?   | Type         |
| agencyId     | Agency identifier         | Int?      | AgencyId     |
| agencyName   | Agency display name       | String?   | AgencyName   |
| likes        | Like count                | Int?      | Likes        |
| isLiked      | User liked this review    | Boolean?  | IsLiked      |
| commentCount | Comment count             | Int?      | CommentCount |
| subject      | Review subject            | String?   | Subject      |
| hiddenCount  | Hidden comment count      | Int?      | HiddenCount  |
| createdTime  | Review creation timestamp | Int?      | CreatedTime  |
| buzzKey      | Buzz key                  | String?   | BuzzKey      |
| appId        | Application identifier    | String?   | AppId        |
| os           | Operating system          | String?   | Os           |
| campaignId   | Campaign identifier       | Int?      | CampaignId   |
| partitionKey | Database partition key    | String?   | PartitionKey |
| rowKey       | Database row key          | String?   | RowKey       |

- Usage

```kotlin
// Suspend version with image
val imageFile = File("/path/to/review-image.jpg")
val result = campaignService.postReviewSuspend(
    campaignId = "12345",
    message = "Great campaign! Highly recommend!",
    image = imageFile
)

// Callback version without image
campaignService.postReview(
    campaignId = "12345",
    message = "Amazing experience, will definitely participate again!",
    image = null
) { result ->
    when (result) {
        is CampaignResult.SuccessPostReview -> {
            // Handle successful review posting
            val postedReview = result.result
            val reviewId = postedReview.rowKey
            val message = postedReview.message
            val createdTime = postedReview.createdTime
            val userId = postedReview.userId
            val campaignId = postedReview.campaignId
            val imageUrl = postedReview.imageUrl

            println("Review posted successfully!")
            println("Review ID: $reviewId")
            println("Message: $message")
            println("Created: $createdTime")
            println("User: $userId")
            println("Campaign: $campaignId")
            if (imageUrl != null) {
                println("Image: $imageUrl")
            }
        }
        is CampaignResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message

            when (errorCode) {
                "REVIEW_ALREADY_EXISTS" -> {
                    // User already reviewed this campaign
                }
                "IMAGE_TOO_LARGE" -> {
                    // Image file size exceeds limit
                }
                "PROFANITY_DETECTED" -> {
                    // Review message contains inappropriate content
                }
            }
        }
    }
}
```

---

### autocompleteCampaign

Provides autocomplete suggestions for campaign search functionality.

- Request (caller-supplied)

| Field Name | Description                     | Mandatory | Data Type |
|------------|---------------------------------|-----------|-----------|
| keyword    | Search keyword for autocomplete | M         | String    |
| top        | Number of suggestions to return | O         | Int?      |
| locale     | Locale identifier               | O         | Int?      |

- Response (`AutocompleteResponse`) [Raw response from API]  
  HTTP status: 200

- Usage

```kotlin
// Suspend
val result = campaignService.autocompleteCampaign(
    keyword = "coff",
    top = 10,
    locale = 1054
)

// Callback
campaignService.autocompleteCampaign("coff", 10, 1054) { result ->
    when (result) {
        is CampaignResult.SuccessAutocompleteRaw -> {
            // Handle successful autocomplete
            val suggestions = result.result
            // Process autocomplete suggestions
        }
        is CampaignResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### verifyCampaign

Verifies campaign eligibility and status for the current user.

- Request (caller-supplied)

| Field Name | Description         | Mandatory | Data Type           |
|------------|---------------------|-----------|---------------------|
| id         | Campaign identifier | M         | String              |
| options    | Additional options  | O         | Map<String, String> |

- Response (`VerifyResponse`) [Raw response from API]  
  HTTP status: 200

- Usage

```kotlin
// Suspend
val result = campaignService.verifyCampaign(
    id = "12345",
    options = mapOf("checkEligibility" to "true")
)

// Callback
campaignService.verifyCampaign("12345", mapOf("strict" to "false")) { result ->
    when (result) {
        is CampaignResult.SuccessVerifyRaw -> {
            // Handle successful verification
            val verifyResponse = result.result
            // Process verification results
        }
        is CampaignResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### chatRedeem

Handles chat-based campaign redemption functionality.

- Request (caller-supplied)

| Field Name | Description        | Mandatory | Data Type           |
|------------|--------------------|-----------|---------------------|
| options    | Redemption options | O         | Map<String, String> |

- Response (`ChatRedeemResponse`) [Raw response from API]  
  HTTP status: 200

- Usage

```kotlin
// Suspend
val result = campaignService.chatRedeem(
    options = mapOf("chatId" to "chat-123")
)

// Callback
campaignService.chatRedeem(mapOf("sessionId" to "session-456")) { result ->
    when (result) {
        is CampaignResult.SuccessChatRedeemRaw -> {
            // Handle successful chat redemption
            val chatRedeemResponse = result.result
            // Process chat redemption results
        }
        is CampaignResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

## CampaignResult Sealed Class

All methods return a `CampaignResult` which is a sealed class with the following possible outcomes:

```kotlin
sealed class CampaignResult {
    data class SuccessCampaignList(val result: List<Campaign>) : CampaignResult()
    data class SuccessCampaignDetail(val result: CampaignDetails) : CampaignResult()
    data class SuccessFavoriteList(val result: List<Campaign>) : CampaignResult()
    data class SuccessFavoriteCampaign(val result: FavoriteResponse) : CampaignResult()
    data class SuccessRedeem(val result: RedeemResponse) : CampaignResult()
    data class SuccessReviewList(val result: List<Review>) : CampaignResult()
    data class SuccessPostReview(val result: Review) : CampaignResult()
    data class SuccessAutocompleteRaw(val result: Any) : CampaignResult()
    data class SuccessVerifyRaw(val result: Any) : CampaignResult()
    data class SuccessChatRedeemRaw(val result: Any) : CampaignResult()
    data class Error(val error: ErrorInfo) : CampaignResult()
}
```

---

## Summary

The CampaignUseCase provides comprehensive campaign management functionality for promotional offers,
deals, and rewards systems. It supports advanced filtering, favorites management, redemption
processing, review systems, search autocomplete, campaign verification, and chat-based redemption.
The rich Campaign entity includes extensive metadata for pricing, availability, categories, and
engagement metrics, making it ideal for building sophisticated e-commerce and loyalty program
applications with full campaign lifecycle management.

All methods are available in both suspend (coroutine) and callback versions for flexible integration
with different architectural patterns. The consistent error handling and response validation ensure
reliable operation across all campaign operations.