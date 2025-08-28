## UserLevelUseCase Guide

This guide shows how to initialize and use every public method in `UserLevelUseCase`, with suspend
and callback examples where available. The UserLevelUseCase provides user level information
functionality for retrieving user level configurations and benefits.

### Getting an instance

```kotlin
val userLevelService = BuzzebeesSDK.instance().userLevelUseCase
```

---

### getLevels

Retrieves user level configurations and information by application ID.

- Request (caller-supplied)

| Field Name | Description            | Mandatory | Data Type |
|------------|------------------------|-----------|-----------|
| appId      | Application identifier | M         | String    |

- Response (`UserLevelResponse`)
  HTTP status: 200

### UserLevelResponse Entity Fields

| Field Name     | Description                      | Data Type          | JSON Field      |
|----------------|----------------------------------|--------------------|-----------------|
| transactionId  | Transaction identifier           | String?            | transaction_id  |
| result         | List of user level items         | List<ResultItem?>? | result          |
| pageNumber     | Current page number              | Int?               | page_number     |
| statusCode     | Response status code             | Int?               | status_code     |
| limit          | Items per page limit             | Int?               | limit           |
| isSuccess      | Success status flag              | Boolean?           | is_success      |
| error          | Error message if any             | String?            | error           |

### ResultItem Entity Fields

| Field Name  | Description                      | Data Type | JSON Field   |
|-------------|----------------------------------|-----------|--------------|
| benefits    | Level benefits description       | String?   | benefits     |
| mission     | Level mission requirements       | String?   | mission      |
| levelName   | User level name                  | String?   | level_name   |
| levelId     | User level identifier            | Int?      | level_id     |
| description | Level description                | String?   | description  |
| sponsorId   | Sponsor identifier               | Int?      | sponsor_id   |

- Usage

```kotlin
// Suspend
val result = userLevelService.getLevels("my_app_id")

// Callback
userLevelService.getLevels("my_app_id") { result ->
    when (result) {
        is UserLevelResult.Success -> {
            // Handle successful user level retrieval
            val levelResponse = result.result

            val isSuccess = levelResponse.isSuccess
            val levels = levelResponse.result

            if (isSuccess == true) {
                levels?.forEach { levelItem ->
                    val levelName = levelItem?.levelName
                    val benefits = levelItem?.benefits
                    val mission = levelItem?.mission
                    val description = levelItem?.description

                    println("Level: $levelName")
                    println("Benefits: $benefits")
                    println("Mission: $mission")
                }
            }
        }
        is UserLevelResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

## Summary

The UserLevelUseCase provides user level information functionality within the Buzzebees SDK. It offers methods to retrieve user level configurations, benefits, and mission requirements by application ID, enabling applications to implement user level systems and reward structures.
