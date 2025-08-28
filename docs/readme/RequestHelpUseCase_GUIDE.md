## RequestHelpUseCase Guide

This guide shows how to initialize and use every public method in `RequestHelpUseCase`, with suspend and callback examples where available. The RequestHelpUseCase provides comprehensive help and support functionality for managing user help requests, comments, and forum-style interactions within the Buzzebees ecosystem.

### Getting an instance

```kotlin
val requestHelpService = BuzzebeesSDK.instance().requestHelpUseCase
```

---

### getHelpCode

Retrieves a help code that can be used to create new help requests. This code is required for the postHelp method.

- Request (caller-supplied)

| Field Name    | Description                | Mandatory | Data Type |
|---------------|----------------------------|-----------|-----------|
| os            | Operating system           | M         | String    |
| platform      | Platform type              | M         | String    |
| clientVersion | Client application version | M         | String    |

- Response (`RequestHelpCode`) [source](../buzzebees_sdk/src/main/java/com/buzzebees/sdk/entity/request_help/RequestHelpCode.kt)  
  HTTP status: 200

### RequestHelpCode Entity Fields

| Field Name | Description       | Data Type | JSON Field |
|------------|-------------------|-----------|------------|
| code       | Help request code | String?   | code       |

- Usage

```kotlin
// Suspend
val result = requestHelpService.getHelpCode("Android", "mobile", "1.0.0")

// Callback
requestHelpService.getHelpCode("Android", "mobile", "1.0.0") { result ->
    when (result) {
        is RequestHelpResult.SuccessHelpCode -> {
            // Handle successful help code retrieval
            val helpCode = result.result.code
            println("Help Code: $helpCode")
        }
        is RequestHelpResult.Error -> {
            // Handle error
            println("Error: ${result.error.message}")
        }
    }
}
```

---

### getHelpList

Retrieves a paginated list of help requests for a specific request ID.

- Request (caller-supplied)

| Field Name | Description                    | Mandatory | Data Type |
|------------|--------------------------------|-----------|-----------|
| requestId  | Request identifier             | M         | String    |
| lastRowKey | Key for pagination (next page) | O         | String?   |

- Response (`List<RequestHelp>`) [source](../buzzebees_sdk/src/main/java/com/buzzebees/sdk/entity/request_help/RequestHelp.kt)  
  HTTP status: 200

### RequestHelp Entity Fields

| Field Name   | Description            | Data Type | JSON Field   |
|--------------|------------------------|-----------|--------------|
| userId       | User identifier        | String?   | UserId       |
| name         | User name              | String?   | Name         |
| message      | Help message content   | String?   | Message      |
| imageUrl     | Image URL if attached  | String?   | ImageUrl     |
| width        | Image width            | Int?      | Width        |
| height       | Image height           | Int?      | Height       |
| photoId      | Photo identifier       | String?   | PhotoId      |
| type         | Request type           | String?   | Type         |
| agencyId     | Agency identifier      | String?   | AgencyId     |
| agencyName   | Agency name            | String?   | AgencyName   |
| likes        | Number of likes        | Int?      | Likes        |
| isLiked      | Whether user liked it  | Boolean?  | IsLiked      |
| commentCount | Number of comments     | Int?      | CommentCount |
| createdTime  | Creation timestamp     | Int?      | CreatedTime  |
| buzzKey      | Unique identifier      | String?   | BuzzKey      |
| campaignId   | Campaign identifier    | String?   | CampaignId   |
| partitionKey | Database partition key | String?   | PartitionKey |
| rowKey       | Database row key       | String?   | RowKey       |

- Usage

```kotlin
// Suspend
val result = requestHelpService.getHelpList("request_123", null)

// Callback
requestHelpService.getHelpList("request_123", null) { result ->
    when (result) {
        is RequestHelpResult.SuccessHelpList -> {
            // Handle successful help list retrieval
            val helpList = result.result
            helpList.forEach { helpItem ->
                println("Help: ${helpItem.message}")
                println("User: ${helpItem.name}")
                println("Likes: ${helpItem.likes}")
            }
        }
        is RequestHelpResult.Error -> {
            // Handle error
            println("Error: ${result.error.message}")
        }
    }
}
```

---

### getHelpDetail

Retrieves detailed information for a specific help request using its buzz key.

- Request (caller-supplied)

| Field Name | Description            | Mandatory | Data Type |
|------------|------------------------|-----------|-----------|
| buzzKey    | Unique help identifier | M         | String    |

- Response (`RequestHelp`) [source](../buzzebees_sdk/src/main/java/com/buzzebees/sdk/entity/request_help/RequestHelp.kt)  
  HTTP status: 200

- Usage

```kotlin
// Suspend
val result = requestHelpService.getHelpDetail("buzz_key_123")

// Callback
requestHelpService.getHelpDetail("buzz_key_123") { result ->
    when (result) {
        is RequestHelpResult.SuccessHelpDetail -> {
            // Handle successful help detail retrieval
            val helpDetail = result.result
            println("Message: ${helpDetail.message}")
            println("User: ${helpDetail.name}")
            println("Likes: ${helpDetail.likes}")
        }
        is RequestHelpResult.Error -> {
            // Handle error
            println("Error: ${result.error.message}")
        }
    }
}
```

---

### postHelp

Creates a new help request with optional image attachment.

- Request (caller-supplied)

| Field Name | Description                         | Mandatory | Data Type |
|------------|-------------------------------------|-----------|-----------|
| requestId  | Request identifier from getHelpCode | M         | String    |
| message    | Help message content                | M         | String    |
| image      | Optional image file attachment      | O         | File?     |

- Response (`RequestHelp`) [source](../buzzebees_sdk/src/main/java/com/buzzebees/sdk/entity/request_help/RequestHelp.kt)  
  HTTP status: 200

- Usage

```kotlin
import java.io.File

// Suspend - With image
val imageFile = File("/path/to/screenshot.jpg")
val result = requestHelpService.postHelp(
    requestId = "request_code_123",
    message = "I need help with my account",
    image = imageFile
)

// Callback
val imageFile = File("/path/to/error_screenshot.png")
requestHelpService.postHelp("request_code_123", "Need help", imageFile) { result ->
    when (result) {
        is RequestHelpResult.SuccessPostHelp -> {
            // Handle successful help post
            val postedHelp = result.result
            println("Help posted: ${postedHelp.message}")
            println("Buzz Key: ${postedHelp.buzzKey}")
            println("Image URL: ${postedHelp.imageUrl}")
        }
        is RequestHelpResult.Error -> {
            // Handle error
            println("Error: ${result.error.message}")
        }
    }
}
```

---

### getCommentList

Retrieves a paginated list of comments for a specific help request.

- Request (caller-supplied)

| Field Name | Description                    | Mandatory | Data Type |
|------------|--------------------------------|-----------|-----------|
| buzzKey    | Help request identifier        | M         | String    |
| lastRowKey | Key for pagination (next page) | O         | String?   |

- Response (`List<Comment>`) [source](../buzzebees_sdk/src/main/java/com/buzzebees/sdk/entity/request_help/Comment.kt)  
  HTTP status: 200

### Comment Entity Fields

| Field Name   | Description            | Data Type | JSON Field   |
|--------------|------------------------|-----------|--------------|
| userId       | User identifier        | String?   | UserId       |
| name         | User name              | String?   | Name         |
| message      | Comment content        | String?   | Message      |
| imageUrl     | Image URL if attached  | String?   | ImageUrl     |
| width        | Image width            | Int?      | Width        |
| height       | Image height           | Int?      | Height       |
| photoId      | Photo identifier       | String?   | PhotoId      |
| type         | Comment type           | String?   | Type         |
| agencyId     | Agency identifier      | String?   | AgencyId     |
| agencyName   | Agency name            | String?   | AgencyName   |
| likes        | Number of likes        | Int?      | Likes        |
| isLiked      | Whether user liked it  | Boolean?  | IsLiked      |
| commentCount | Number of replies      | Int?      | CommentCount |
| createdTime  | Creation timestamp     | Int?      | CreatedTime  |
| buzzKey      | Parent help identifier | String?   | BuzzKey      |
| campaignId   | Campaign identifier    | String?   | CampaignId   |
| partitionKey | Database partition key | String?   | PartitionKey |
| rowKey       | Database row key       | String?   | RowKey       |

- Usage

```kotlin
// Suspend
val result = requestHelpService.getCommentList("buzz_key_123", null)

// Callback
requestHelpService.getCommentList("buzz_key_123", null) { result ->
    when (result) {
        is RequestHelpResult.SuccessCommentList -> {
            // Handle successful comment list retrieval
            val comments = result.result
            comments.forEach { comment ->
                println("${comment.name}: ${comment.message}")
                println("Likes: ${comment.likes}")
            }
        }
        is RequestHelpResult.Error -> {
            // Handle error
            println("Error: ${result.error.message}")
        }
    }
}
```

---

### postComment

Adds a new comment to a help request with optional image attachment.

- Request (caller-supplied)

| Field Name | Description                    | Mandatory | Data Type |
|------------|--------------------------------|-----------|-----------|
| buzzKey    | Help request identifier        | M         | String    |
| message    | Comment content                | M         | String    |
| image      | Optional image file attachment | O         | File?     |

- Response (`Comment`) [source](../buzzebees_sdk/src/main/java/com/buzzebees/sdk/entity/request_help/Comment.kt)  
  HTTP status: 200

- Usage

```kotlin
import java.io.File

// Suspend - With image
val imageFile = File("/path/to/solution_screenshot.jpg")
val result = requestHelpService.postComment(
    buzzKey = "buzz_key_123",
    message = "Here's what I did to fix it",
    image = imageFile
)

// Callback
val screenshotFile = File("/path/to/helpful_screenshot.png")
requestHelpService.postComment("buzz_key_123", "Thanks!", screenshotFile) { result ->
    when (result) {
        is RequestHelpResult.SuccessPostComment -> {
            // Handle successful comment post
            val postedComment = result.result
            println("Comment posted: ${postedComment.message}")
            println("Image URL: ${postedComment.imageUrl}")
        }
        is RequestHelpResult.Error -> {
            // Handle error
            println("Error: ${result.error.message}")
        }
    }
}
```

---

### linkCampaign (Like)

Likes a help request or comment, showing appreciation for the content.

- Request (caller-supplied)

| Field Name | Description                | Mandatory | Data Type |
|------------|----------------------------|-----------|-----------|
| buzzKey    | Help or comment identifier | M         | String    |

- Response (`LikeForumResponse`) [source](../buzzebees_sdk/src/main/java/com/buzzebees/sdk/entity/request_help/LikeForumResponse.kt)  
  HTTP status: 200

### LikeForumResponse Entity Fields

| Field Name | Description           | Data Type | JSON Field |
|------------|-----------------------|-----------|------------|
| result     | Like operation result | Boolean?  | result     |

- Usage

```kotlin
// Suspend
val result = requestHelpService.linkCampaign("buzz_key_123")

// Callback
requestHelpService.linkCampaign("buzz_key_123") { result ->
    when (result) {
        is RequestHelpResult.SuccessLikeUnLike -> {
            // Handle successful like
            println("Liked successfully!")
        }
        is RequestHelpResult.Error -> {
            // Handle error
            println("Error: ${result.error.message}")
        }
    }
}
```

---

### unLikeCampaign (Unlike)

Unlikes a previously liked help request or comment.

- Request (caller-supplied)

| Field Name | Description                | Mandatory | Data Type |
|------------|----------------------------|-----------|-----------|
| buzzKey    | Help or comment identifier | M         | String    |

- Response (`LikeForumResponse`) [source](../buzzebees_sdk/src/main/java/com/buzzebees/sdk/entity/request_help/LikeForumResponse.kt)  
  HTTP status: 200

- Usage

```kotlin
// Suspend
val result = requestHelpService.unLikeCampaign("buzz_key_123")

// Callback
requestHelpService.unLikeCampaign("buzz_key_123") { result ->
    when (result) {
        is RequestHelpResult.SuccessLikeUnLike -> {
            // Handle successful unlike
            println("Unliked successfully!")
        }
        is RequestHelpResult.Error -> {
            // Handle error
            println("Error: ${result.error.message}")
        }
    }
}
```

---

## Summary

The RequestHelpUseCase provides help and support functionality within the Buzzebees SDK. It offers eight methods for managing help requests:

- **getHelpCode**: Generate request codes for creating help requests
- **getHelpList**: Retrieve paginated lists of help requests
- **getHelpDetail**: Get detailed information for specific help requests
- **postHelp**: Create new help requests with optional image attachments
- **getCommentList**: Retrieve paginated comments for help requests
- **postComment**: Add comments to help requests with optional image attachments
- **linkCampaign**: Like help requests or comments
- **unLikeCampaign**: Unlike help requests or comments

Each method supports both suspend and callback patterns with comprehensive error handling.
