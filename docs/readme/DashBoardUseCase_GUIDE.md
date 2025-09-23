## DashBoardUseCase Guide

This guide shows how to initialize and use every public method in `DashBoardUseCase`, with suspend
and callback examples where available. The DashBoardUseCase provides comprehensive dashboard
content management functionality for retrieving both main application dashboards and specific
sub-dashboards with localization support.

### Getting an instance

```kotlin
val dashboardService = BuzzebeesSDK.instance().dashBoardUseCase
```

---

### getCachedMainDashboard

Retrieves cached main dashboard from local storage without making network requests.

- Request (caller-supplied)

No parameters required.

- Response (`DashBoardResult.SuccessMain`)
  Returns cached main dashboard or empty list

- Usage

```kotlin
// Synchronous call - no network request
val result = dashboardService.getCachedMainDashboard()

when (result) {
    is DashBoardResult.SuccessMain -> {
        val cachedDashboard = result.result
        if (cachedDashboard.isNotEmpty()) {
            // Use cached data
            cachedDashboard.forEach { item ->
                println("Cached Item: ${item.name}")
            }
        } else {
            // No cached data available - consider calling getMainDashboard()
            println("No cached main dashboard available")
        }
    }
    is DashBoardResult.Error -> {
        // Handle error (unlikely for cache operation)
    }
}
```

---

### getMainDashboard

Retrieves the main application dashboard with localized content and dashboard items.

- Request (caller-supplied)

| Field Name | Description                           | Mandatory | Data Type |
|------------|---------------------------------------|-----------|-----------|
| appName    | Application name from Backoffice      | M         | String    |
| locale     | User language (1054: Thai, 1033: Eng) | M         | Int       |

- Response (`List<Dashboard>`)
  HTTP status: 200

### Dashboard Entity Fields

| Field Name         | Description                   | Data Type        | JSON Field         |
|--------------------|-------------------------------|------------------|--------------------|
| type               | Dashboard item type           | String?          | type               |
| size               | Dashboard item size           | String?          | size               |
| menu               | Menu identifier               | String?          | menu               |
| subCampaignDetails | Nested dashboard items        | `List<Dashboard>?` | subcampaigndetails |
| catHeaderTh        | Category header in Thai       | String?          | cat_header_th      |
| catHeaderEn        | Category header in English    | String?          | cat_header_en      |
| id                 | Dashboard item identifier     | String?          | id                 |
| imageUrl           | Dashboard item image URL      | String?          | image_url          |
| url                | Dashboard item action URL     | String?          | url                |
| line1              | First line of text content    | String?          | line1              |
| line2              | Second line of text content   | String?          | line2              |
| line3              | Third line of text content    | String?          | line3              |
| line4              | Fourth line of text content   | String?          | line4              |
| cat                | Category identifier           | String?          | cat                |
| andNs              | Android namespace             | String?          | and_ns             |
| gaLabel            | Google Analytics label        | String?          | ga_label           |
| iconUrl            | Icon image URL                | String?          | icon_url           |
| gaCampaignName     | GA campaign name              | String?          | ga_campaign_name   |
| gaCategoryName     | GA category name              | String?          | ga_category_name   |
| headerThai         | Header text in Thai           | String?          | header_thai        |
| headerEn           | Header text in English        | String?          | header_en          |
| gaName             | Google Analytics name         | String?          | ga_name            |
| config             | Configuration identifier      | String?          | config             |
| name               | Item name in default language | String?          | name               |
| nameEN             | Item name in English          | String?          | nameEN             |
| endDate            | End date timestamp            | Long?            | end_date           |
| createBy           | Creator name                  | String?          | createBy           |
| createByEN         | Creator name in English       | String?          | createByEN         |
| rowKey             | Database row key              | String?          | RowKey             |
| partitionKey       | Database partition key        | String?          | PartitionKey       |
| startTime          | Start time string             | String?          | start_time         |
| startDate          | Start date timestamp          | Long?            | start_date         |
| hashtags           | Associated hashtags           | String?          | hashtags           |

- Usage

```kotlin
// Suspend
val result = dashboardService.getMainDashboard(
    appName = "MyApp",
    locale = 1054 // Thai locale
)

// Callback
dashboardService.getMainDashboard(
    appName = "MyApp",
    locale = 1033 // English locale
) { result ->
    when (result) {
        is DashBoardResult.SuccessMain -> {
            // Handle successful main dashboard retrieval
            val dashboardItems = result.result

            dashboardItems.forEach { item ->
                val itemId = item.id
                val itemType = item.type
                val itemSize = item.size
                val itemName = item.name
                val itemNameEn = item.nameEN
                val imageUrl = item.imageUrl
                val iconUrl = item.iconUrl
                val actionUrl = item.url

                println("Dashboard Item: $itemName (ID: $itemId)")
                println("Type: $itemType, Size: $itemSize")

                // Handle localized content
                val displayName = if (locale == 1033) itemNameEn ?: itemName else itemName
                val headerText = if (locale == 1033) item.headerEn else item.headerThai

                println("Display Name: $displayName")
                println("Header: $headerText")

                // Handle text content lines
                listOfNotNull(item.line1, item.line2, item.line3, item.line4).let { lines ->
                    if (lines.isNotEmpty()) {
                        println("Content Lines:")
                        lines.forEachIndexed { index, line ->
                            println("  Line ${index + 1}: $line")
                        }
                    }
                }

                // Handle nested sub-campaign details
                item.subCampaignDetails?.let { subItems ->
                    println("Sub-items count: ${subItems.size}")
                    subItems.forEach { subItem ->
                        println("  Sub-item: ${subItem.name} (Type: ${subItem.type})")
                    }
                }
            }
        }
        is DashBoardResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message

            when (errorCode) {
                "APP_NOT_FOUND" -> {
                    // Handle application not found
                }
                "INVALID_LOCALE" -> {
                    // Handle invalid locale
                }
                "NO_DASHBOARD_CONTENT" -> {
                    // Handle no dashboard content available
                }
            }
        }
    }
}
```

---

### getCachedSubDashboard

Retrieves cached sub-dashboard from local storage without making network requests.

- Request (caller-supplied)

| Field Name | Description                     | Mandatory | Data Type |
|------------|---------------------------------|-----------|-----------|
| appName    | Application name for cache key  | M         | String    |

- Response (`DashBoardResult.SuccessSub`)
  Returns cached sub-dashboard or empty list

- Usage

```kotlin
// Synchronous call - no network request
val result = dashboardService.getCachedSubDashboard(appName = "MyApp")

when (result) {
    is DashBoardResult.SuccessSub -> {
        val cachedSubDashboard = result.result
        if (cachedSubDashboard.isNotEmpty()) {
            // Use cached data
            cachedSubDashboard.forEach { item ->
                println("Cached Sub-item: ${item.name}")
            }
        } else {
            // No cached data available - consider calling getSubDashboard()
            println("No cached sub-dashboard available for $appName")
        }
    }
    is DashBoardResult.Error -> {
        // Handle error (unlikely for cache operation)
    }
}
```

---

### getSubDashboard

Retrieves a specific sub-dashboard by dashboard name with localized content.

- Request (caller-supplied)

| Field Name | Description                           | Mandatory | Data Type |
|------------|---------------------------------------|-----------|-----------|
| appName    | Application name from Backoffice      | M         | String    |
| locale     | User language (1054: Thai, 1033: Eng) | M         | Int       |

- Response (`List<Dashboard>`)
  HTTP status: 200

### Dashboard Entity Fields

| Field Name         | Description                   | Data Type        | JSON Field         |
|--------------------|-------------------------------|------------------|--------------------|
| type               | Dashboard item type           | String?          | type               |
| size               | Dashboard item size           | String?          | size               |
| menu               | Menu identifier               | String?          | menu               |
| subCampaignDetails | Nested dashboard items        | `List<Dashboard>?` | subcampaigndetails |
| catHeaderTh        | Category header in Thai       | String?          | cat_header_th      |
| catHeaderEn        | Category header in English    | String?          | cat_header_en      |
| id                 | Dashboard item identifier     | String?          | id                 |
| imageUrl           | Dashboard item image URL      | String?          | image_url          |
| url                | Dashboard item action URL     | String?          | url                |
| line1              | First line of text content    | String?          | line1              |
| line2              | Second line of text content   | String?          | line2              |
| line3              | Third line of text content    | String?          | line3              |
| line4              | Fourth line of text content   | String?          | line4              |
| cat                | Category identifier           | String?          | cat                |
| andNs              | Android namespace             | String?          | and_ns             |
| gaLabel            | Google Analytics label        | String?          | ga_label           |
| iconUrl            | Icon image URL                | String?          | icon_url           |
| gaCampaignName     | GA campaign name              | String?          | ga_campaign_name   |
| gaCategoryName     | GA category name              | String?          | ga_category_name   |
| headerThai         | Header text in Thai           | String?          | header_thai        |
| headerEn           | Header text in English        | String?          | header_en          |
| gaName             | Google Analytics name         | String?          | ga_name            |
| config             | Configuration identifier      | String?          | config             |
| name               | Item name in default language | String?          | name               |
| nameEN             | Item name in English          | String?          | nameEN             |
| endDate            | End date timestamp            | Long?            | end_date           |
| createBy           | Creator name                  | String?          | createBy           |
| createByEN         | Creator name in English       | String?          | createByEN         |
| rowKey             | Database row key              | String?          | RowKey             |
| partitionKey       | Database partition key        | String?          | PartitionKey       |
| startTime          | Start time string             | String?          | start_time         |
| startDate          | Start date timestamp          | Long?            | start_date         |
| hashtags           | Associated hashtags           | String?          | hashtags           |

- Usage

```kotlin
// Suspend
val result = dashboardService.getSubDashboard(
    appName = "promotions",
    locale = 1054
)

// Callback
dashboardService.getSubDashboard(
    appName = "rewards_dashboard",
    locale = 1033
) { result ->
    when (result) {
        is DashBoardResult.SuccessSub -> {
            // Handle successful sub-dashboard retrieval
            val dashboardItems = result.result

            println("Sub-dashboard loaded with ${dashboardItems.size} items")

            dashboardItems.forEach { item ->
                val itemType = item.type
                val itemSize = item.size
                val itemName = item.name
                val config = item.config
                val category = item.cat

                println("Item: $itemName")
                println("Type: $itemType, Size: $itemSize")
                println("Config: $config, Category: $category")

                // Handle date information
                val startDate = item.startDate
                val endDate = item.endDate
                if (startDate != null && endDate != null) {
                    println("Active from: $startDate to $endDate")
                }

                // Handle analytics data
                val gaLabel = item.gaLabel
                val gaCampaignName = item.gaCampaignName
                val gaCategoryName = item.gaCategoryName

                if (gaLabel != null) {
                    println("Analytics: $gaLabel")
                    println("  Campaign: $gaCampaignName")
                    println("  Category: $gaCategoryName")
                }

                // Handle hashtags
                item.hashtags?.let { hashtags ->
                    println("Hashtags: $hashtags")
                }
            }
        }
        is DashBoardResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message

            when (errorCode) {
                "DASHBOARD_NOT_FOUND" -> {
                    // Handle dashboard not found
                }
                "ACCESS_DENIED" -> {
                    // Handle access denied
                }
                "INVALID_DASHBOARD_NAME" -> {
                    // Handle invalid dashboard name
                }
            }
        }
    }
}
```

---

## Summary

The DashBoardUseCase provides comprehensive dashboard content management functionality for dynamic
application interfaces within the Buzzebees SDK. It offers both main application dashboards and
specific sub-dashboards with full localization support, nested content structures, and rich metadata
for analytics and content management systems.