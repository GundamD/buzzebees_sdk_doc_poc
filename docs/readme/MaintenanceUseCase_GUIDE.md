## MaintenanceUseCase Guide

This guide shows how to initialize and use every public method in `MaintenanceUseCase`, with suspend
and callback examples where available. The MaintenanceUseCase provides maintenance status
information
for application downtime and service maintenance notifications.

### Getting an instance

```kotlin
val maintenanceService = BuzzebeesSDK.instance().maintenanceUseCase
```

---

### getMaintenance

Retrieves maintenance status information by URL.

- Request (caller-supplied)

| Field Name | Description              | Mandatory | Data Type |
|------------|--------------------------|-----------|-----------|
| url        | Maintenance endpoint URL | M         | String    |

- Response (`Maintenance`)
  HTTP status: 200

### Maintenance Entity Fields

| Field Name          | Description                   | Data Type | JSON Field            |
|---------------------|-------------------------------|-----------|-----------------------|
| headerPrimary       | Primary header text           | String?   | header_primary        |
| buttonTextPrimary   | Primary button text           | String?   | button_text_primary   |
| headerSecondary     | Secondary header text         | String?   | header_secondary      |
| messagePrimary      | Primary message content       | String?   | message_primary       |
| messageSecondary    | Secondary message content     | String?   | message_secondary     |
| enabled             | Maintenance mode enabled flag | Boolean?  | enabled               |
| buttonTextSecondary | Secondary button text         | String?   | button_text_secondary |
| landingUrl          | Landing page URL              | String?   | landing_url           |

- Usage

```kotlin
// Suspend
val result = maintenanceService.getMaintenance("https://api.example.com/maintenance")

// Callback
maintenanceService.getMaintenance("https://api.example.com/maintenance") { result ->
    when (result) {
        is MaintenanceResult.Success -> {
            // Handle successful maintenance info retrieval
            val maintenance = result.result

            val isEnabled = maintenance.enabled
            val primaryHeader = maintenance.headerPrimary
            val primaryMessage = maintenance.messagePrimary
            val landingUrl = maintenance.landingUrl

            println("Maintenance enabled: $isEnabled")
            println("Header: $primaryHeader")
            println("Message: $primaryMessage")
        }
        is MaintenanceResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

### getMaintenance

Retrieves maintenance status information by application ID.

- Request (caller-supplied)

| Field Name | Description            | Mandatory | Data Type |
|------------|------------------------|-----------|-----------|
| -          | None. SDK supplies application info automatically. | -         | -         |

- Response (`Maintenance`)
  HTTP status: 200

### Maintenance Entity Fields

| Field Name          | Description                   | Data Type | JSON Field            |
|---------------------|-------------------------------|-----------|-----------------------|
| headerPrimary       | Primary header text           | String?   | header_primary        |
| buttonTextPrimary   | Primary button text           | String?   | button_text_primary   |
| headerSecondary     | Secondary header text         | String?   | header_secondary      |
| messagePrimary      | Primary message content       | String?   | message_primary       |
| messageSecondary    | Secondary message content     | String?   | message_secondary     |
| enabled             | Maintenance mode enabled flag | Boolean?  | enabled               |
| buttonTextSecondary | Secondary button text         | String?   | button_text_secondary |
| landingUrl          | Landing page URL              | String?   | landing_url           |

- Usage

```kotlin
// Suspend
val result = maintenanceService.getMaintenance()

// Callback
maintenanceService.getMaintenance() { result ->
    when (result) {
        is MaintenanceResult.Success -> {
            // Handle successful maintenance info retrieval
            val maintenance = result.result

            val isEnabled = maintenance.enabled
            val primaryHeader = maintenance.headerPrimary
            val primaryMessage = maintenance.messagePrimary

            println("Maintenance enabled: $isEnabled")
            println("Header: $primaryHeader")
            println("Message: $primaryMessage")
        }
        is MaintenanceResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message
        }
    }
}
```

---

## Summary

The MaintenanceUseCase provides maintenance status information functionality within the Buzzebees
SDK. It offers two methods to retrieve maintenance information either by URL or application ID,
helping applications handle service maintenance and downtime scenarios with appropriate user
messaging.
