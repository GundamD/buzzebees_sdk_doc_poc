## CartUseCase Guide

This guide shows how to initialize and use every public method in `CartUseCase`, with suspend
and callback examples where available. The CartUseCase provides comprehensive shopping cart
management functionality including adding items to cart, retrieving cart count, and generating
shopping cart URLs for WebView integration.

### Getting an instance

```kotlin
val cartService = BuzzebeesSDK.instance().cart
```

---

### addCart

Adds a campaign item to the user's shopping cart with specified parameters. Automatically generates
a shopping cart URL on success for immediate checkout flow.

- Request (caller-supplied)

| Field Name       | Description                                              | Mandatory | Data Type | Default |
|------------------|----------------------------------------------------------|-----------|-----------|---------|
| id               | Campaign identifier                                      | M         | String    | -       |
| mode             | Add mode (e.g., "add")                                   | M         | String    | -       |
| qty              | Quantity to add                                          | M         | String    | -       |
| sideCampaignJson | Side campaign configuration in JSON                      | O         | String    | ""      |
| clearCart        | Whether to clear cart before adding (for immediate checkout) | O     | Boolean   | true    |

- Response (`CartResult.SuccessAddCart`)

| Field Name | Description                        | Data Type          |
|------------|------------------------------------|--------------------|
| data       | Cart count response                | CartCountResponse  |
| cartUrl    | Shopping cart URL for WebView      | String?            |

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
    qty = "1"
)

when (result) {
    is CartResult.SuccessAddCart -> {
        val totalItems = result.data.cartCount
        val cartUrl = result.cartUrl
        
        println("Item added successfully!")
        println("Total items in cart: $totalItems")
        
        // Open cart in WebView
        cartUrl?.let { url ->
            openWebView(url)
        }
    }
    is CartResult.Error -> {
        println("Error: ${result.error.message}")
    }
}

// Callback
cartService.addCart(
    id = "12345",
    mode = "add",
    qty = "2",
    sideCampaignJson = """{"color": "red", "size": "M"}""",
    clearCart = true
) { result ->
    when (result) {
        is CartResult.SuccessAddCart -> {
            val totalItems = result.data.cartCount
            val cartUrl = result.cartUrl

            println("Item added successfully!")
            println("Total items in cart: $totalItems")

            // Open cart in WebView for checkout
            cartUrl?.let { url ->
                openWebView(url)
            }
        }
        is CartResult.Error -> {
            val errorCode = result.error.code
            val errorMessage = result.error.message

            println("Failed to add to cart: $errorMessage (code: $errorCode)")
        }
    }
}
```

---

### getCartCount

Retrieves the total number of items currently in the user's shopping cart.

- Request (caller-supplied)

No parameters required.

- Response (`CartResult.SuccessCartCount`)

| Field Name | Description         | Data Type         |
|------------|---------------------|-------------------|
| data       | Cart count response | CartCountResponse |

### CartCountResponse Entity Fields

| Field Name | Description                   | Data Type | JSON Field |
|------------|-------------------------------|-----------|------------|
| cartCount  | Total number of items in cart | Long?     | cart_count |

- Usage

```kotlin
// Suspend
val result = cartService.getCartCount()

when (result) {
    is CartResult.SuccessCartCount -> {
        val totalItems = result.data.cartCount ?: 0
        println("Total items in cart: $totalItems")
        
        // Update UI badge
        updateCartBadge(totalItems)
    }
    is CartResult.Error -> {
        println("Failed to get cart count: ${result.error.message}")
    }
}

// Callback
cartService.getCartCount { result ->
    when (result) {
        is CartResult.SuccessCartCount -> {
            val totalItems = result.data.cartCount ?: 0

            println("Cart count retrieved successfully!")
            println("Total items in cart: $totalItems")

            // Update UI badge or counter
            updateCartBadge(totalItems)
        }
        is CartResult.Error -> {
            val errorMessage = result.error.message
            println("Failed to get cart count: $errorMessage")
        }
    }
}
```

---

### getCartUrl

Generates a shopping cart URL for opening in WebView. Supports different cart types
for various shopping flows.

- Request (caller-supplied)

| Field Name | Description                                      | Mandatory | Data Type | Default |
|------------|--------------------------------------------------|-----------|-----------|---------|
| cartType   | Cart type (e.g., "MarketPlacePrivilege")         | O         | String?   | null    |

- Response (`CartResult.SuccessCartUrl`)

| Field Name | Description                   | Data Type |
|------------|-------------------------------|-----------|
| url        | Shopping cart URL for WebView | String    |

- Usage

```kotlin
// Suspend - Default cart
val result = cartService.getCartUrl()

when (result) {
    is CartResult.SuccessCartUrl -> {
        val cartUrl = result.url
        println("Cart URL: $cartUrl")
        
        // Open in WebView
        openWebView(cartUrl)
    }
    is CartResult.Error -> {
        println("Failed to get cart URL: ${result.error.message}")
    }
}

// Suspend - MarketPlace Privilege cart
val marketplaceResult = cartService.getCartUrl(cartType = "MarketPlacePrivilege")

when (marketplaceResult) {
    is CartResult.SuccessCartUrl -> {
        openWebView(marketplaceResult.url)
    }
    is CartResult.Error -> {
        println("Error: ${marketplaceResult.error.message}")
    }
}

// Callback
cartService.getCartUrl { result ->
    when (result) {
        is CartResult.SuccessCartUrl -> {
            val cartUrl = result.url

            println("Cart URL retrieved successfully!")
            println("URL: $cartUrl")

            // Open cart page in WebView
            openWebView(cartUrl)
        }
        is CartResult.Error -> {
            val errorMessage = result.error.message
            println("Failed to get cart URL: $errorMessage")
        }
    }
}

// Callback - With cart type
cartService.getCartUrl(cartType = "MarketPlacePrivilege") { result ->
    when (result) {
        is CartResult.SuccessCartUrl -> {
            openWebView(result.url)
        }
        is CartResult.Error -> {
            println("Error: ${result.error.message}")
        }
    }
}
```

---

## CartResult Types

The CartUseCase returns sealed class `CartResult` with the following types:

| Result Type        | Description                              | Fields                    |
|--------------------|------------------------------------------|---------------------------|
| SuccessAddCart     | Item added to cart successfully          | data, cartUrl             |
| SuccessCartCount   | Cart count retrieved successfully        | data                      |
| SuccessCartUrl     | Cart URL generated successfully          | url                       |
| Error              | Operation failed                         | error (ErrorResponse)     |

---

## Common Usage Patterns

### Pattern 1: Add to Cart and Checkout

```kotlin
// Add item and immediately open cart for checkout
suspend fun addToCartAndCheckout(campaignId: String, quantity: Int) {
    val result = cartService.addCart(
        id = campaignId,
        mode = "add",
        qty = quantity.toString(),
        clearCart = true  // Clear cart for single-item checkout
    )
    
    when (result) {
        is CartResult.SuccessAddCart -> {
            result.cartUrl?.let { url ->
                openWebView(url)
            }
        }
        is CartResult.Error -> {
            showError(result.error.message)
        }
    }
}
```

### Pattern 2: Update Cart Badge

```kotlin
// Fetch and update cart badge on app launch or after login
suspend fun refreshCartBadge() {
    when (val result = cartService.getCartCount()) {
        is CartResult.SuccessCartCount -> {
            val count = result.data.cartCount?.toInt() ?: 0
            cartBadgeView.text = if (count > 0) count.toString() else ""
            cartBadgeView.isVisible = count > 0
        }
        is CartResult.Error -> {
            // Silently fail or show default state
            cartBadgeView.isVisible = false
        }
    }
}
```

### Pattern 3: Open Cart Page

```kotlin
// Open cart page when user taps cart icon
fun onCartIconClick() {
    lifecycleScope.launch {
        when (val result = cartService.getCartUrl()) {
            is CartResult.SuccessCartUrl -> {
                openWebView(result.url)
            }
            is CartResult.Error -> {
                showToast("Unable to open cart")
            }
        }
    }
}
```

### Pattern 4: Legacy Callback Style

```kotlin
// For non-coroutine projects
class CartManager {
    
    fun addToCart(campaignId: String, onResult: (Boolean, String?) -> Unit) {
        cartService.addCart(
            id = campaignId,
            mode = "add",
            qty = "1"
        ) { result ->
            when (result) {
                is CartResult.SuccessAddCart -> {
                    onResult(true, result.cartUrl)
                }
                is CartResult.Error -> {
                    onResult(false, null)
                }
            }
        }
    }
}
```

---

## Summary

The CartUseCase provides essential shopping cart functionality for e-commerce applications built
with the Buzzebees SDK. Key features include:

- **Add to Cart** - Add items with automatic cart URL generation for seamless checkout
- **Cart Count** - Retrieve current cart item count for badge updates
- **Cart URL** - Generate authenticated shopping cart URLs for WebView integration
- **Dual API Support** - Both suspend functions and callbacks for flexible integration

All methods require user authentication. Ensure the user is logged in before calling cart operations.
