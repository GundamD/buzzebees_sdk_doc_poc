## CategoryUseCase Guide

This guide shows how to initialize and use every public method in `CategoryUseCase`, with suspend
and callback examples where available. The CategoryUseCase provides comprehensive category
management functionality for retrieving hierarchical campaign categories with optional
configuration and localization support.

### Getting an instance

```kotlin
val categoryService = BuzzebeesSDK.instance().categoryUseCase
```

---

### getCategory

Retrieves a list of campaign categories with hierarchical structure and optional filtering by configuration and locale.

- Request (caller-supplied)

| Field Name    | Description                                  | Mandatory | Data Type |
|---------------|----------------------------------------------|-----------|-----------|
| byConfig      | Filter categories by configuration           | M         | Boolean   |
| config        | Configuration identifier for campaign list   | M         | String    |
| deviceLocale  | Device locale identifier                     | O         | Int?      |
| locale        | Locale identifier for content localization   | O         | Int?      |

- Response (`List<Category>`) [source](../buzzebees_sdk/src/main/java/com/buzzebees/sdk/entity/category/Category.kt)  
  HTTP status: 200

### Category Entity Fields

| Field Name   | Description                        | Data Type       | JSON Field   |
|--------------|------------------------------------|-----------------|--------------|
| mode         | Category display mode              | String?         | mode         |
| listConfig   | List configuration settings        | String?         | list_config  |
| name         | Category display name              | String?         | name         |
| id           | Category unique identifier         | Int?            | id           |
| detail       | Category detailed description      | String?         | detail       |
| count        | Number of campaigns in category    | Int?            | count        |
| imageUrl     | Category image URL                 | String?         | image_url    |
| subCats      | List of subcategories              | List<Category>? | subcats      |

- Usage

```kotlin
// Suspend
val result = categoryService.getCategory(
    byConfig = true,
    config = "main_categories",
    deviceLocale = 1054,
    locale = 1033
)

// Callback
categoryService.getCategory(
    byConfig = true,
    config = "rewards_categories",
    deviceLocale = 1054,
    locale = null
) { result ->
    when (result) {
        is CategoryResult.SuccessCategoryList -> {
            // Handle successful category list retrieval
            val categories = result.result
            
            categories.forEach { category ->
                val categoryName = category.name
                val categoryId = category.id
                val campaignCount = category.count
                val imageUrl = category.imageUrl
                val subcategories = category.subCats
                
                println("Category: $categoryName (ID: $categoryId)")
                println("Campaigns: $campaignCount")
                
                // Handle subcategories if present
                subcategories?.forEach { subcat ->
                    println("  Subcategory: ${subcat.name} (${subcat.count} campaigns)")
                }
            }
        }
        is CategoryResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
            
            when (errorCode) {
                "CONFIG_NOT_FOUND" -> {
                    // Handle configuration not found
                }
                "INVALID_LOCALE" -> {
                    // Handle invalid locale
                }
                "NO_CATEGORIES_FOUND" -> {
                    // Handle no categories available
                }
            }
        }
    }
}
```

---

## Summary

The CategoryUseCase provides essential category management functionality for organizing and discovering campaigns within the Buzzebees SDK. It offers hierarchical category browsing with configuration-based filtering and localization support for building category navigation systems in e-commerce and rewards applications.