## NotificationUseCase Guide

This guide shows how to initialize and use every public method in `NotificationUseCase`, with suspend
and callback examples where available. The NotificationUseCase provides notification management
functionality for retrieving and marking notifications as read.

### Getting an instance

```kotlin
val notificationService = BuzzebeesSDK.instance().notificationUseCase
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
  HTTP status: 200

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
