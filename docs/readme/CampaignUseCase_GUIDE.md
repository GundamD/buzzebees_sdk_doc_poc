## CampaignUseCase Guide

This guide shows how to initialize and use every public method in `CampaignUseCase`, with suspend
and callback examples where available. The CampaignUseCase provides comprehensive campaign
management functionality including browsing, favorites, and review systems for
promotional campaigns and offers.

### Getting an instance

```kotlin
val campaignService = BuzzebeesSDK.instance().campaignUseCase
```

---

### getCachedList

Retrieves cached campaign list for a specific category without making a network request. This method provides fast access to previously loaded campaigns.

- Request (caller-supplied)

| Field Name | Description        | Mandatory | Data Type |
|------------|--------------------|-----------|-----------|
| category   | Category filter    | O         | String?   |

- Response (`List<Campaign>`)
  HTTP status: 200

> **Campaign Fields Reference**
>
> Same as getCampaignList response - see the complete Campaign entity fields table below.

- Usage

```kotlin
// Suspend
val result = campaignService.getCachedList("food")

// Callback
campaignService.getCachedList("electronics") { result ->
    when (result) {
        is CampaignResult.SuccessCampaignList -> {
            // Handle successful cached list retrieval
            val cachedCampaigns = result.result
            cachedCampaigns.forEach { campaign ->
                val name = campaign.name
                val points = campaign.pointPerUnit
                println("Cached Campaign: $name - Points: $points")
            }
        }
        is CampaignResult.Error -> {
            // Handle error (rarely occurs with cached data)
            val errorMessage = result.error.message
        }
    }
}
```

**Note**: This method returns cached data only. If no cached data exists for the specified category, it returns an empty list. Use `getCampaignList` to fetch fresh data from the server.

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

- Response (`List<Campaign>`)
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

### getFavoriteList

Retrieves a list of campaigns marked as favorites by the user.

- Request (caller-supplied)

| Field Name | Description               | Mandatory | Data Type           |
|------------|---------------------------|-----------|---------------------|
| skip       | Number of items to skip   | O         | Int?                |
| top        | Number of items to return | O         | Int?                |
| locale     | Locale identifier         | O         | Int?                |
| options    | Additional options        | O         | Map<String, String> |

- Response (`List<Campaign>`)
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

- Response (`FavoriteResponse`)
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

- Response (`FavoriteResponse`)
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

### reviewList

Retrieves reviews and ratings for a specific campaign with pagination support.

- Request (caller-supplied)

| Field Name | Description                 | Mandatory | Data Type |
|------------|-----------------------------|-----------|-----------|
| campaignId | Campaign identifier         | M         | String    |
| lastRowKey | Last row key for pagination | O         | String?   |

- Response (`List<Review>`)
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

- Response (`Review`) 
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
deals, and rewards systems. It supports advanced filtering, favorites management,
review systems, search autocomplete, campaign verification, and chat-based redemption.
The rich Campaign entity includes extensive metadata for pricing, availability, categories, and
engagement metrics, making it ideal for building sophisticated e-commerce and loyalty program
applications.

All methods are available in both suspend (coroutine) and callback versions for flexible integration
with different architectural patterns. The consistent error handling and response validation ensure
reliable operation across all campaign operations.

**Note**: For campaign detail and redemption operations, see the separate CampaignDetailUseCase and RedeemUseCase documentation.

---
