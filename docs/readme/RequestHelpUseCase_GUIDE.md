## RequestHelpUseCase Guide

This guide shows how to initialize and use every public method in `RequestHelpUseCase`, with suspend
and callback examples where available. The RequestHelpUseCase provides comprehensive help forum functionality
for posting help requests, viewing help listings, commenting on posts, and managing likes/unlikes
for community support features.

### Getting an instance

```kotlin
val requestHelpService = BuzzebeesSDK.instance().requestHelpUseCase
```

---

### getHelpCode

Retrieves a help request code for creating new help requests.

- Request (caller-supplied)

No parameters required.

- Response (`RequestHelpCode`)
  HTTP status: 200

### RequestHelpCode Entity Fields

| Field Name | Description           | Data Type | JSON Field |
|------------|-----------------------|-----------|------------|
| code       | Help request code     | String?   | code       |

- Usage

```kotlin
// Suspend
val result = requestHelpService.getHelpCode()

// Callback
requestHelpService.getHelpCode { result ->
    when (result) {
        is RequestHelpResult.SuccessHelpCode -> {
            // Handle successful help code retrieval
            val helpCode = result.result
            println("Help code: ${helpCode.code}")
        }
        is RequestHelpResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### getHelpList

Retrieves a paginated list of help requests.

- Request (caller-supplied)

| Field Name | Description                      | Mandatory | Data Type |
|------------|----------------------------------|-----------|-----------|
| requestId  | Help request ID or category code | M         | String    |
| lastRowKey | Row key for pagination           | O         | String?   |

- Response (`List<RequestHelp>`)
  HTTP status: 200

### RequestHelp Entity Fields

| Field Name   | Description                 | Data Type | JSON Field   |
|--------------|----------------------------|-----------|--------------|
| userId       | User ID of poster          | String?   | UserId       |
| name         | Name of poster             | String?   | Name         |
| message      | Help request message       | String?   | Message      |
| imageUrl     | Attached image URL         | String?   | ImageUrl     |
| width        | Image width                | Int?      | Width        |
| height       | Image height               | Int?      | Height       |
| photoId      | Photo identifier           | String?   | PhotoId      |
| type         | Request type               | String?   | Type         |
| agencyId     | Agency identifier          | String?   | AgencyId     |
| agencyName   | Agency name                | String?   | AgencyName   |
| likes        | Number of likes            | Int?      | Likes        |
| isLiked      | Current user liked flag    | Boolean?  | IsLiked      |
| commentCount | Number of comments         | Int?      | CommentCount |
| createdTime  | Creation timestamp         | Int?      | CreatedTime  |
| buzzKey      | Unique buzz key            | String?   | BuzzKey      |
| campaignId   | Related campaign ID        | String?   | CampaignId   |
| partitionKey | Database partition key     | String?   | PartitionKey |
| rowKey       | Database row key           | String?   | RowKey       |

- Usage

```kotlin
// Suspend
val result = requestHelpService.getHelpList("HELP_GENERAL", null)

// Callback
requestHelpService.getHelpList("HELP_GENERAL", null) { result ->
    when (result) {
        is RequestHelpResult.SuccessHelpList -> {
            // Handle successful help list retrieval
            val helpRequests = result.result
            helpRequests.forEach { helpRequest ->
                println("Help: ${helpRequest.name} - ${helpRequest.message}")
                println("Likes: ${helpRequest.likes}, Comments: ${helpRequest.commentCount}")
            }
        }
        is RequestHelpResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### getHelpDetail

Retrieves detailed information for a specific help request.

- Request (caller-supplied)

| Field Name | Description      | Mandatory | Data Type |
|------------|------------------|-----------|-----------|
| buzzKey    | Help request key | M         | String    |

- Response (`RequestHelp`)
  HTTP status: 200

- Usage

```kotlin
// Suspend
val result = requestHelpService.getHelpDetail("help_buzz_key_123")

// Callback
requestHelpService.getHelpDetail("help_buzz_key_123") { result ->
    when (result) {
        is RequestHelpResult.SuccessHelpDetail -> {
            // Handle successful help detail retrieval
            val helpDetail = result.result
            println("Help from: ${helpDetail.name}")
            println("Message: ${helpDetail.message}")
            println("Likes: ${helpDetail.likes}, Comments: ${helpDetail.commentCount}")
        }
        is RequestHelpResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### postHelp

Posts a new help request with optional image attachment.

- Request (caller-supplied)

| Field Name | Description             | Mandatory | Data Type |
|------------|-------------------------|-----------|-----------|
| requestId  | Help request ID/code    | M         | String    |
| message    | Help request message    | M         | String    |
| image      | Optional image file     | O         | File?     |

- Response (`RequestHelp`)
  HTTP status: 200

- Usage

```kotlin
// Suspend
val result = requestHelpService.postHelp(
    requestId = "HELP_CODE_123",
    message = "I need help with account login issues",
    image = File("/path/to/screenshot.jpg")
)

// Callback
requestHelpService.postHelp(
    requestId = "HELP_CODE_123",
    message = "I need help with account login issues",
    image = null
) { result ->
    when (result) {
        is RequestHelpResult.SuccessPostHelp -> {
            // Handle successful help post
            val postedHelp = result.result
            println("Help posted successfully!")
            println("Buzz key: ${postedHelp.buzzKey}")
        }
        is RequestHelpResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### getCommentList

Retrieves a paginated list of comments for a specific help request.

- Request (caller-supplied)

| Field Name | Description             | Mandatory | Data Type |
|------------|-------------------------|-----------|-----------|
| buzzKey    | Help request key        | M         | String    |
| lastRowKey | Row key for pagination  | O         | String?   |

- Response (`List<Comment>`)
  HTTP status: 200

### Comment Entity Fields

| Field Name   | Description                 | Data Type | JSON Field   |
|--------------|----------------------------|-----------|--------------|
| userId       | User ID of commenter       | String?   | UserId       |
| name         | Name of commenter          | String?   | Name         |
| message      | Comment message            | String?   | Message      |
| imageUrl     | Attached image URL         | String?   | ImageUrl     |
| width        | Image width                | Int?      | Width        |
| height       | Image height               | Int?      | Height       |
| photoId      | Photo identifier           | String?   | PhotoId      |
| type         | Comment type               | String?   | Type         |
| agencyId     | Agency identifier          | String?   | AgencyId     |
| agencyName   | Agency name                | String?   | AgencyName   |
| likes        | Number of likes            | Int?      | Likes        |
| isLiked      | Current user liked flag    | Boolean?  | IsLiked      |
| commentCount | Number of replies          | Int?      | CommentCount |
| createdTime  | Creation timestamp         | Int?      | CreatedTime  |
| buzzKey      | Parent help request key    | String?   | BuzzKey      |
| campaignId   | Related campaign ID        | String?   | CampaignId   |
| partitionKey | Database partition key     | String?   | PartitionKey |
| rowKey       | Database row key           | String?   | RowKey       |

- Usage

```kotlin
// Suspend
val result = requestHelpService.getCommentList("help_buzz_key_123", null)

// Callback
requestHelpService.getCommentList("help_buzz_key_123", null) { result ->
    when (result) {
        is RequestHelpResult.SuccessCommentList -> {
            // Handle successful comment list retrieval
            val comments = result.result
            comments.forEach { comment ->
                println("Comment by: ${comment.name}")
                println("Message: ${comment.message}")
            }
        }
        is RequestHelpResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### postComment

Posts a comment on a help request with optional image attachment.

- Request (caller-supplied)

| Field Name | Description         | Mandatory | Data Type |
|------------|---------------------|-----------|-----------|
| buzzKey    | Help request key    | M         | String    |
| message    | Comment message     | M         | String    |
| image      | Optional image file | O         | File?     |

- Response (`Comment`)
  HTTP status: 200

- Usage

```kotlin
// Suspend
val result = requestHelpService.postComment(
    buzzKey = "help_buzz_key_123",
    message = "Try resetting your password from settings",
    image = null
)

// Callback
requestHelpService.postComment(
    buzzKey = "help_buzz_key_123",
    message = "Try resetting your password from settings",
    image = null
) { result ->
    when (result) {
        is RequestHelpResult.SuccessPostComment -> {
            // Handle successful comment post
            val postedComment = result.result
            println("Comment posted successfully!")
            println("Message: ${postedComment.message}")
        }
        is RequestHelpResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### linkCampaign

Likes a help request (adds a like/thumbs up).

- Request (caller-supplied)

| Field Name | Description      | Mandatory | Data Type |
|------------|------------------|-----------|-----------|
| buzzKey    | Help request key | M         | String    |

- Response (`LikeForumResponse`) 
  HTTP status: 200

### LikeForumResponse Entity Fields

| Field Name | Description    | Data Type | JSON Field |
|------------|----------------|-----------|------------|
| result     | Like result    | Boolean?  | result     |

- Usage

```kotlin
// Suspend
val result = requestHelpService.linkCampaign("help_buzz_key_123")

// Callback
requestHelpService.linkCampaign("help_buzz_key_123") { result ->
    when (result) {
        is RequestHelpResult.SuccessLikeUnLike -> {
            // Handle successful like
            val likeResponse = result.result
            if (likeResponse.result == true) {
                println("Successfully liked the help request!")
            }
        }
        is RequestHelpResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### unLikeCampaign

Unlikes a help request (removes a like/thumbs up).

- Request (caller-supplied)

| Field Name | Description      | Mandatory | Data Type |
|------------|------------------|-----------|-----------|
| buzzKey    | Help request key | M         | String    |

- Response (`LikeForumResponse`)
  HTTP status: 200

- Usage

```kotlin
// Suspend
val result = requestHelpService.unLikeCampaign("help_buzz_key_123")

// Callback
requestHelpService.unLikeCampaign("help_buzz_key_123") { result ->
    when (result) {
        is RequestHelpResult.SuccessLikeUnLike -> {
            // Handle successful unlike
            val unlikeResponse = result.result
            if (unlikeResponse.result == true) {
                println("Successfully unliked the help request!")
            }
        }
        is RequestHelpResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

## Summary

The RequestHelpUseCase provides comprehensive help forum functionality within the Buzzebees SDK. It enables users to create help requests, browse help listings, engage with community content through comments and likes, and manage forum interactions. The UseCase supports image attachments and pagination for efficient data loading.
