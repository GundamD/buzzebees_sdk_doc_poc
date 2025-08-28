## PointLogUseCase Guide

This guide shows how to initialize and use every public method in `PointLogUseCase`, with suspend
and callback examples where available. The PointLogUseCase provides point transaction history
functionality for retrieving user point earning and spending logs.

### Getting an instance

```kotlin
val pointLogService = BuzzebeesSDK.instance().pointLogUseCase
```

---

### getPointLog

Retrieves point transaction logs filtered by month and type with pagination support.

- Request (caller-supplied)

| Field Name | Description                   | Mandatory | Data Type |
|------------|-------------------------------|-----------|-----------|
| month      | Month filter (YYYY-MM format) | M         | String    |
| type       | Transaction type filter       | M         | String    |
| lastRowKey | Last row key for pagination   | O         | String?   |

- Response (`List<PointLog>`)
  HTTP status: 200

### PointLog Entity Fields

| Field Name   | Description                         | Data Type | JSON Field   |
|--------------|-------------------------------------|-----------|--------------|
| rowKey       | Database row key identifier         | String?   | RowKey       |
| userId       | User identifier                     | String?   | UserId       |
| info         | Additional information              | String?   | Info         |
| detail       | Transaction detail description      | String?   | Detail       |
| message      | Transaction message                 | String?   | Message      |
| points       | Point amount (positive or negative) | Int?      | Points       |
| type         | Transaction type                    | String?   | Type         |
| timestamp    | Transaction timestamp               | Long?     | Timestamp    |
| campaignName | Related campaign name               | String?   | CampaignName |
| campaignId   | Related campaign identifier         | String?   | CampaignId   |
| input        | Input data for transaction          | String?   | Input        |

- Usage

```kotlin
// Suspend
val result = pointLogService.getPointLog(
    month = "2024-08",
    type = "earn",
    lastRowKey = null
)

// Callback
pointLogService.getPointLog(
    month = "2024-08",
    type = "spend",
    lastRowKey = null
) { result ->
    when (result) {
        is PointLogResult.SuccessPointLog -> {
            // Handle successful point log retrieval
            val pointLogs = result.result

            pointLogs.forEach { pointLog ->
                val points = pointLog.points
                val message = pointLog.message
                val campaignName = pointLog.campaignName
                val timestamp = pointLog.timestamp

                println("Points: $points")
                println("Message: $message")
                println("Campaign: $campaignName")
            }
        }
        is PointLogResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

## Summary

The PointLogUseCase provides point transaction history functionality within the Buzzebees SDK. It
offers methods to retrieve user point earning and spending logs filtered by month and transaction
type, with pagination support for handling large datasets of point transaction history.
