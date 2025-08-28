## CartUseCase Guide

This guide shows how to initialize and use every public method in `CartUseCase`, with suspend
and callback examples where available. The CartUseCase provides comprehensive shopping cart
management functionality including adding items to cart, retrieving cart count, and managing
access tokens for cart operations.

### Getting an instance

```kotlin
val cartService = BuzzebeesSDK.instance().cartUseCase
```

---

### addCart

Adds a campaign item to the user's shopping cart with specified parameters.

- Request (caller-supplied)

| Field Name       | Description                         | Mandatory | Data Type |
|------------------|-------------------------------------|-----------|-----------|
| id               | Campaign identifier                 | M         | String    |
| mode             | Add mode (e.g., "add", "replace")   | M         | String    |
| qty              | Quantity to add                     | M         | String    |
| sideCampaignJson | Side campaign configuration in JSON | M         | String    |

- Response (`CartCountResponse`) [source](../buzzebees_sdk/src/main/java/com/buzzebees/sdk/entity/cart/CartCountResponse.kt)  
  HTTP status: 200

### CartCountResponse Entity Fields

| Field Name | Description                   | Data Type | JSON Field |
|------------|-------------------------------|-----------|------------|
| cartCount  | Total number of items in cart | Long?     | cart_count |

- Usage

```kotlin
// Suspend
val result = cartService.addCart(
    id = "12345",
    mode = "add",
    qty = "2",
    sideCampaignJson = """{"color": "red", "size": "M"}"""
)

// Callback
cartService.addCart("12345", "add", "1", "{}") { result ->
    when (result) {
        is CartResult.SuccessAddCart -> {
            // Handle successful item addition
            val cartResponse = result.result
            val totalItems = cartResponse.cartCount

            println("Item added successfully!")
            println("Total items in cart: $totalItems")
        }
        is CartResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message

            when (errorCode) {
                "CAMPAIGN_NOT_FOUND" -> {
                    // Handle campaign not found
                }
                "INSUFFICIENT_QUANTITY" -> {
                    // Handle insufficient quantity
                }
                "CAMPAIGN_EXPIRED" -> {
                    // Handle expired campaign
                }
                "CART_LIMIT_EXCEEDED" -> {
                    // Handle cart limit exceeded
                }
            }
        }
    }
}
```

---

### getCartCount

Retrieves the total number of items currently in the user's shopping cart.

- Request (caller-supplied)

No parameters required.

- Response (`CartCountResponse`) [source](../buzzebees_sdk/src/main/java/com/buzzebees/sdk/entity/cart/CartCountResponse.kt)  
  HTTP status: 200

### CartCountResponse Entity Fields

| Field Name | Description                   | Data Type | JSON Field |
|------------|-------------------------------|-----------|------------|
| cartCount  | Total number of items in cart | Long?     | cart_count |

- Usage

```kotlin
// Suspend
val result = cartService.getCartCount()

// Callback
cartService.getCartCount { result ->
    when (result) {
        is CartResult.SuccessCartCount -> {
            // Handle successful cart count retrieval
            val cartResponse = result.result
            val totalItems = cartResponse.cartCount

            println("Cart count retrieved successfully!")
            println("Total items in cart: $totalItems")

            // Update UI badge or counter
            updateCartBadge(totalItems ?: 0)
        }
        is CartResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message

            println("Failed to get cart count: $errorMessage")
        }
    }
}
```

---

### getAccessToken

Retrieves an access token for cart operations using provided configuration data.

- Request (caller-supplied)

| Field Name | Description                       | Mandatory | Data Type |
|------------|-----------------------------------|-----------|-----------|
| dataJson   | Configuration data in JSON format | M         | String    |

- Response (`AccessToken`) [source](../buzzebees_sdk/src/main/java/com/buzzebees/sdk/entity/cart/AccessToken.kt)  
  HTTP status: 200

### AccessToken Entity Fields

| Field Name | Description                 | Data Type | JSON Field |
|------------|-----------------------------|-----------|------------|
| data       | Token data container        | Data?     | data       |
| success    | Operation success indicator | Boolean?  | success    |

### Data Entity Fields

| Field Name | Description  | Data Type | JSON Field |
|------------|--------------|-----------|------------|
| key        | Access token | String?   | key        |

- Usage

```kotlin
// Prepare configuration data
val configData = """{
    "clientId": "your_client_id",
    "scope": "cart_access",
    "permissions": ["read", "write"]
}"""

// Suspend
val result = cartService.getAccessToken(configData)

// Callback
cartService.getAccessToken(configData) { result ->
    when (result) {
        is CartResult.SuccessAccessToken -> {
            // Handle successful access token retrieval
            val accessTokenResponse = result.result
            val isSuccess = accessTokenResponse.success
            val tokenKey = accessTokenResponse.data?.key

            if (isSuccess == true && tokenKey != null) {
                println("Access token retrieved successfully!")
                println("Token: $tokenKey")

                // Store token for future cart operations
                storeAccessToken(tokenKey)
            } else {
                println("Token retrieval succeeded but no valid token returned")
            }
        }
        is CartResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message

            when (errorCode) {
                "INVALID_CLIENT" -> {
                    // Handle invalid client credentials
                }
                "INSUFFICIENT_PERMISSIONS" -> {
                    // Handle insufficient permissions
                }
                "TOKEN_GENERATION_FAILED" -> {
                    // Handle token generation failure
                }
            }
        }
    }
}
```

---

## Summary

The CartUseCase provides essential shopping cart functionality for e-commerce applications built
with the Buzzebees SDK. It offers comprehensive cart management including item addition, cart
tracking, and access control with both suspend and callback API patterns for flexible integration
across different architectural approaches.