## NotificationUseCase Guide

This guide shows how to initialize and use every public method in `NotificationUseCase`, with suspend
and callback examples where available. The NotificationUseCase provides notification management
functionality for retrieving and marking notifications as read.

### Getting an instance

```kotlin
val notificationService = BuzzebeesSDK.instance().notificationUseCase
```

---

### getCacheNotificationList

Retrieves cached notifications from local storage for offline access.

- Request (caller-supplied)

No parameters required.

- Response (`List<Notification>`)
  Returns cached notifications or empty list

- Usage

```kotlin
// Synchronous call - no network request
val cachedNotifications = notificationService.getCacheNotificationList()

if (cachedNotifications.isNotEmpty()) {
    // Use cached data
    cachedNotifications.forEach { notification ->
        val objectType = notification.objectType
        val isRead = notification.isRead
        val createTime = notification.createTime
        
        println("Cached Notification: $objectType, Read: $isRead")
    }
} else {
    // No cached data available - consider calling getNotificationList()
    println("No cached notifications available")
}
```

---

### getNotificationList

Retrieves a list of notifications with pagination and sorting options.

- Request (caller-supplied)

| Field Name | Description                          | Mandatory | Data Type |
|------------|--------------------------------------|-----------|-----------|
| skip       | Number of records to skip            | O         | Int?      |
| top        | Maximum number of records to return  | O         | Int?      |
| sortBy     | Sort field and direction             | O         | String?   |

- Response (`List<Notification>`)

### Notification Entity Fields

| Field Name              | Description                          | Data Type            | JSON Field           |
|-------------------------|--------------------------------------|----------------------|----------------------|
| objectType              | Type of notification object          | String?              | ObjectType           |
| objectCategoryId        | Category identifier                  | Int?                 | ObjectCategoryId     |
| objectCampaignId        | Campaign identifier                  | Int?                 | ObjectCampaignId     |
| isRead                  | Read status flag                     | Boolean?             | IsRead               |
| createTime              | Creation timestamp                   | Long?                | CreateTime           |
| expireDate              | Expiration timestamp                 | Long?                | ExpireDate           |
| notificationObject      | Notification content object          | NotificationObject?  | Object               |
| partitionKey            | Database partition key               | String?              | PartitionKey         |
| rowKey                  | Database row key                     | String?              | RowKey               |
| imagePath               | Notification image path              | String?              | ImagePath            |
| notificationRawObject   | Raw notification data                | Map<String, Any>?    | -                    |

- Usage

```kotlin
// Suspend
val result = notificationService.getNotificationList(
    skip = 0,
    top = 20,
    sortBy = "createdate_desc"
)

// Callback
notificationService.getNotificationList(
    skip = 0,
    top = 20,
    sortBy = "createdate_desc"
) { result ->
    when (result) {
        is NotificationResult.SuccessList -> {
            // Handle successful notification list retrieval
            val notifications = result.result

            notifications.forEach { notification ->
                val objectType = notification.objectType
                val isRead = notification.isRead
                val createTime = notification.createTime
                val expireDate = notification.expireDate
                val imagePath = notification.imagePath

                println("Type: $objectType, Read: $isRead")
                println("Created: $createTime")
            }
        }
        is NotificationResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### getNotificationCount

Retrieves the total count of user notifications for badge display and management.

- Request (caller-supplied)

No parameters required.

- Response (`Int`) - Total notification count
  HTTP status: 200

- Usage

```kotlin
// Suspend
val result = notificationService.getNotificationCount()

// Callback
notificationService.getNotificationCount { result ->
    when (result) {
        is NotificationResult.SuccessCount -> {
            // Handle successful notification count retrieval
            val notificationCount = result.count
            
            println("Total notifications: $notificationCount")
            
            // Update badge or UI element
            if (notificationCount > 0) {
                // Show badge with count
                updateNotificationBadge(notificationCount)
            } else {
                // Hide badge
                hideNotificationBadge()
            }
        }
        is NotificationResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
            
            when (errorCode) {
                "NO_NOTIFICATIONS" -> {
                    // Handle no notifications case
                }
                "AUTHENTICATION_REQUIRED" -> {
                    // Handle authentication required
                }
            }
        }
    }
}
```

---

### readNotification

Marks notifications as read by providing notification IDs.

- Request (caller-supplied)

| Field Name | Description                           | Mandatory | Data Type |
|------------|---------------------------------------|-----------|-----------|
| ids        | Comma-separated notification IDs      | M         | String    |

- Response (`NoContentResponse`)
  HTTP status: 200

- Usage

```kotlin
// Suspend
val result = notificationService.readNotification("id1,id2,id3")

// Callback
notificationService.readNotification("notification_id_123") { result ->
    when (result) {
        is NotificationResult.SuccessRead -> {
            // Handle successful notification read
            val response = result.result
            println("Notifications marked as read")
        }
        is NotificationResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

## Summary

The NotificationUseCase provides notification management functionality within the Buzzebees SDK. It offers methods to retrieve notification lists with pagination and sorting capabilities, and to mark notifications as read, enabling effective notification management in applications.
