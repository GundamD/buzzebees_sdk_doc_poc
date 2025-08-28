## BadgeUseCase Guide

This guide shows how to initialize and use every public method in `BadgeUseCase`, with suspend and
callback examples where available. The BadgeUseCase provides badge and achievement management
functionality for gamification and user engagement systems.

### Getting an instance

```kotlin
val badgeService = BuzzebeesSDK.instance().badgeUseCase
```

---

### getBadgeList

Retrieves a list of badges and achievements for a specific badge identifier. This method is used to
fetch user progress, achievement status, and badge information for gamification systems.

- Request (caller-supplied)

| Field Name | Description             | Mandatory | Data Type |
|------------|-------------------------|-----------|-----------|
| badgeId    | Unique badge identifier | M         | String    |

- Response (`List<Badge>`)
  HTTP status: 200

> **Badge Fields Reference**
>
> See the complete Badge entity fields table below for all available properties including missions,
> progress tracking, and reward information.

- Usage

```kotlin
// Suspend
val result = badgeService.getBadgeList("user-badge-123")

// Callback
badgeService.getBadgeList("user-badge-123") { result ->
    when (result) {
        is BadgeResult.SuccessBadge -> {
            // Handle successful badge list retrieval
            val badges = result.result
            badges.forEach { badge ->
                val name = badge.name
                val description = badge.description
                val points = badge.points
                val level = badge.level
                val isObtained = badge.isObtain
                val percentage = badge.percentage

                println("Badge: $name - Level: $level, Points: $points")
                println("Progress: ${percentage}%, Obtained: $isObtained")

                // Check missions progress
                badge.missions?.forEach { mission ->
                    val missionName = mission?.name
                    val current = mission?.current
                    val target = mission?.value
                    val completed = mission?.isCompleted
                    println("Mission: $missionName - $current/$target (Completed: $completed)")
                }
            }
        }
        is BadgeResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

#### Error Handling

| Error Code | Error ID | Scenario                 | User Message                       | Recommended Action                       |
|------------|----------|--------------------------|------------------------------------|------------------------------------------|
| 404        | -        | Badge not found          | "Badge not found"                  | Show empty state or suggest other badges |
| 403        | -        | No permission to view    | "No permission to view this badge" | Hide badge or show unlock conditions     |
| 409        | -        | Badge conditions not met | "Badge conditions not met yet"     | Show required conditions                 |

#### Implementation Example

```kotlin
badgeService.getBadgeList(badgeId) { result ->
    when (result) {
        is BadgeResult.SuccessBadge -> {
            displayBadges(result.result)
        }
        is BadgeResult.Error -> {
            val action = when (result.error.code) {
                "404" -> ErrorAction.ShowBadgeNotFound()
                "403" -> ErrorAction.ShowBadgeNoPermission()
                "409" -> ErrorAction.ShowBadgeConditionsNotMet()
                else -> ErrorAction.ShowGenericError(result.error.message)
            }
            handleErrorAction(action)
        }
    }
}
```

---

## Entity Fields Reference

### Badge Entity

The Badge entity represents achievements, rewards, and gamification elements with comprehensive
progress tracking and configuration options:

| Field Name                   | Description                              | Data Type       | JSON Field                   |
|------------------------------|------------------------------------------|-----------------|------------------------------|
| **Basic Information**        |                                          |                 |                              |
| name                         | Badge display name                       | String?         | Name                         |
| description                  | Badge description                        | String?         | Description                  |
| fBDescription                | Facebook share description               | String?         | FBDescription                |
| points                       | Points awarded for badge                 | Int?            | Points                       |
| level                        | Current badge level                      | Int?            | Level                        |
| maxLevels                    | Maximum achievable levels                | Int?            | MaxLevels                    |
| **Status & Configuration**   |                                          |                 |                              |
| active                       | Badge is active/available                | Boolean?        | Active                       |
| deleted                      | Badge is deleted                         | Boolean?        | Deleted                      |
| isObtain                     | User has obtained this badge             | Boolean?        | IsObtain                     |
| obtainOn                     | When badge was obtained                  | Any?            | ObtainOn                     |
| percentage                   | Progress percentage towards badge        | Double?         | Percentage                   |
| **Time Management**          |                                          |                 |                              |
| startDate                    | Badge availability start date            | Long?           | StartDate                    |
| endDate                      | Badge availability end date              | Long?           | EndDate                      |
| period                       | Badge validity period                    | String?         | Period                       |
| userLevelExpireDate          | User level expiration date               | Long?           | UserLevelExpireDate          |
| userLevelExpireIn            | Expiration time value                    | Int?            | UserLevelExpireIn            |
| userLevelExpirePeriod        | Expiration period unit                   | String?         | UserLevelExpirePeriod        |
| userLevelExpireInRounding    | Round expiration time                    | Boolean?        | UserLevelExpireInRounding    |
| **Notification Settings**    |                                          |                 |                              |
| isSkipNoti                   | Skip notifications for this badge        | Boolean?        | IsSkipNoti                   |
| isSkipNotiWhenNotEarnPoints  | Skip notifications when no points earned | Boolean?        | IsSkipNotiWhenNotEarnPoints  |
| isSkipNotiAutoRedeem         | Skip notifications for auto-redeem       | Boolean?        | IsSkipNotiAutoRedeem         |
| pushNotiMessage              | Push notification message                | Any?            | PushNotiMessage              |
| lineMessageSetting           | LINE message configuration               | Any?            | LineMessageSetting           |
| nextNotificationDate         | Next notification timestamp              | Long?           | NextNotificationDate         |
| nextNotificationPeriod       | Next notification period                 | String?         | NextNotificationPeriod       |
| nextNotification             | Next notification details                | Any?            | NextNotification             |
| nextNotificationRounding     | Round next notification time             | Boolean?        | NextNotificationRounding     |
| **Auto-Redeem System**       |                                          |                 |                              |
| autoRedeemCampaignId         | Campaign ID for auto-redeem              | Int?            | AutoRedeemCampaignId         |
| autoRedeemBadgeLevel         | Badge level trigger for auto-redeem      | Int?            | AutoRedeemBadgeLevel         |
| autoRedeemContinueEveryLevel | Continue auto-redeem at every level      | Boolean?        | AutoRedeemContinueEveryLevel |
| redeemMedia                  | Redemption media configuration           | Any?            | RedeemMedia                  |
| **Dependencies & Grouping**  |                                          |                 |                              |
| dependencyBadge              | Required badge dependencies              | Any?            | DependencyBadge              |
| dependencyBadgeType          | Type of badge dependency                 | Any?            | DependencyBadgeType          |
| grouping                     | Badge group identifier                   | String?         | Grouping                     |
| isSpecific                   | Badge is user-specific                   | Boolean?        | IsSpecific                   |
| **Campaign Integration**     |                                          |                 |                              |
| pushCampaignId               | Associated push campaign ID              | Any?            | PushCampaignId               |
| pushCampaignMessage          | Push campaign message                    | Any?            | PushCampaignMessage          |
| **System Fields**            |                                          |                 |                              |
| agencyId                     | Agency identifier                        | Int?            | AgencyId                     |
| appId                        | Application identifier                   | String?         | AppId                        |
| partitionKey                 | Database partition key                   | String?         | PartitionKey                 |
| rowKey                       | Database row key                         | String?         | RowKey                       |
| timestamp                    | Record timestamp                         | Int?            | Timestamp                    |
| eTag                         | Entity tag for caching                   | String?         | ETag                         |
| **Progress & Missions**      |                                          |                 |                              |
| missions                     | List of associated missions              | List<Mission?>? | Missions                     |
| sequence                     | Badge sequence order                     | Int?            | Sequence                     |
| reset                        | Badge can be reset                       | Boolean?        | Reset                        |
| refreshFromTraceProfile      | Refresh from user trace profile          | Boolean?        | RefreshFromTraceProfile      |
| **Level Management**         |                                          |                 |                              |
| userLevel                    | Current user level                       | Long?           | UserLevel                    |
| **Additional Data**          |                                          |                 |                              |
| customInfo                   | Custom badge information                 | Any?            | CustomInfo                   |

### Mission Entity

The Mission entity represents individual tasks or objectives within a badge:

| Field Name  | Description                  | Data Type | JSON Field  |
|-------------|------------------------------|-----------|-------------|
| tracePlanId | Unique mission trace plan ID | String?   | TracePlanId |
| value       | Target value to achieve      | Int?      | Value       |
| current     | Current progress value       | Int?      | Current     |
| isCompleted | Mission completion status    | Boolean?  | IsCompleted |
| name        | Mission display name         | String?   | Name        |

---

## Summary

The BadgeUseCase provides comprehensive gamification and achievement management capabilities. It
supports complex badge systems with multi-level progression, mission tracking, auto-redeem
functionality, and flexible notification settings. The rich Badge entity includes extensive metadata
for customization, progress tracking, and integration with rewards systems, making it ideal for
building engaging user experiences that drive long-term retention and engagement.