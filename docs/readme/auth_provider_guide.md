# AuthProvider Usage Guide

This guide provides a formal explanation of all available functions in `AuthProvider` along with example usage in Kotlin. It is intended for customers integrating the SDK.

---

## Accessing the AuthProvider

To access the `AuthProvider` instance from the SDK:

```kotlin
val auth =   BuzzebeesSDK.instance().authProvider()
```

This grants access to all authentication and token management functions.

---

## Saving Tokens

**saveAuthToken(token: String)**: Saves the BZBS authentication token.

```kotlin
auth.saveAuthToken("BZBS_TOKEN_VALUE")
```

**saveJwtToken(token: String)**: Saves the JWT token.

```kotlin
auth.saveJwtToken("JWT_TOKEN_VALUE")
```

**saveEWalletToken(token: String)**: Saves the e-wallet token.

```kotlin
auth.saveEWalletToken("EWALLET_TOKEN_VALUE")
```

**saveWalletToken(token: String)**: Saves the wallet token.

```kotlin
auth.saveWalletToken("WALLET_TOKEN_VALUE")
```

**saveFcmToken(token: String)**: Saves the FCM push notification token.

```kotlin
auth.saveFcmToken("FCM_TOKEN_VALUE")
```

**saveLocale(locale: Int)**: Saves the user's locale preference.

```kotlin
auth.saveLocale(1054) // Thai locale
auth.saveLocale(1033) // English locale
```

---

## Retrieving Tokens

**getAuthToken()**: Retrieves the BZBS authentication token.

```kotlin
val bzbsToken = auth.getAuthToken()
```

**getJwtToken()**: Retrieves the JWT token.

```kotlin
val jwtToken = auth.getJwtToken()
```

**getEWalletToken()**: Retrieves the e-wallet token.

```kotlin
val eWalletToken = auth.getEWalletToken()
```

**getWalletToken()**: Retrieves the wallet token.

```kotlin
val walletToken = auth.getWalletToken()
```

**getFcmToken()**: Retrieves the FCM token.

```kotlin
val fcmToken = auth.getFcmToken()
```

**getLocale()**: Retrieves the saved locale preference.

```kotlin
val locale = auth.getLocale() // Returns Int? (1054 for Thai, 1033 for English, etc.)
```

---

## Removing Tokens

**removeAuthToken()**: Removes the BZBS authentication token.

```kotlin
auth.removeAuthToken()
```

**removeJwtToken()**: Removes the JWT token.

```kotlin
auth.removeJwtToken()
```

**removeEWalletToken()**: Removes the e-wallet token.

```kotlin
auth.removeEWalletToken()
```

**removeWalletToken()**: Removes the wallet token.

```kotlin
auth.removeWalletToken()
```

**removeFcmToken()**: Removes the FCM token.

```kotlin
auth.removeFcmToken()
```

**removeLocale()**: Removes the saved locale preference.

```kotlin
auth.removeLocale()
```

**clear()**: Removes all stored tokens and locale.

```kotlin
auth.clear()
```

---

## Updating Multiple Tokens

**updateTokens(...)**: Updates multiple tokens and locale simultaneously. Pass `null` to skip updating a specific value.

```kotlin
auth.updateTokens(
    authToken = "NEW_BZBS_TOKEN",
    jwtToken = "NEW_JWT_TOKEN",
    eWalletToken = null,
    walletToken = "NEW_WALLET_TOKEN",
    fcmToken = null,
    locale = 1054 // Thai locale
)
```

---

## Checking Token Existence

**hasAuthToken()**: Returns true if the BZBS authentication token exists.

```kotlin
if (auth.hasAuthToken()) {
    println("BZBS token exists")
}
```

**hasJwtToken()**: Returns true if the JWT token exists.

```kotlin
if (auth.hasJwtToken()) {
    println("JWT token exists")
}
```

**hasEWalletToken()**: Returns true if the e-wallet token exists.

```kotlin
if (auth.hasEWalletToken()) {
    println("E-Wallet token exists")
}
```

**hasWalletToken()**: Returns true if the wallet token exists.

```kotlin
if (auth.hasWalletToken()) {
    println("Wallet token exists")
}
```

**hasFcmToken()**: Returns true if the FCM token exists.

```kotlin
if (auth.hasFcmToken()) {
    println("FCM token exists")
}
```

**hasLocale()**: Returns true if the locale preference exists.

```kotlin
if (auth.hasLocale()) {
    println("Locale preference exists")
}
```

**hasAnyToken()**: Returns true if at least one token exists.

```kotlin
if (auth.hasAnyToken()) {
    println("At least one token exists")
}
```

**hasToken()**: Returns true if authentication token exists (alias for hasAuthToken).

```kotlin
if (auth.hasToken()) {
    println("Authentication token exists")
}
```

---

## Retrieving Token Information

**getTokenInfo()**: Returns a `TokenInfo` object containing the existence and length of each token.

```kotlin
data class TokenInfo(
    val hasAuthToken: Boolean,
    val hasJwtToken: Boolean,
    val hasEWalletToken: Boolean,
    val hasWalletToken: Boolean,
    val hasFcmToken: Boolean,
    val authTokenLength: Int,
    val jwtTokenLength: Int,
    val eWalletTokenLength: Int,
    val walletTokenLength: Int,
    val fcmTokenLength: Int
)

val tokenInfo = auth.getTokenInfo()

println("Has BZBS Token? ${tokenInfo.hasAuthToken}, Length: ${tokenInfo.authTokenLength}")
println("Has JWT Token? ${tokenInfo.hasJwtToken}, Length: ${tokenInfo.jwtTokenLength}")
println("Has E-Wallet Token? ${tokenInfo.hasEWalletToken}, Length: ${tokenInfo.eWalletTokenLength}")
println("Has Wallet Token? ${tokenInfo.hasWalletToken}, Length: ${tokenInfo.walletTokenLength}")
println("Has FCM Token? ${tokenInfo.hasFcmToken}, Length: ${tokenInfo.fcmTokenLength}")

// Check locale separately
val locale = auth.getLocale()
val hasLocale = auth.hasLocale()
println("Has Locale? $hasLocale, Value: $locale")
```

The `TokenInfo` object includes the following properties:

* `hasAuthToken`: Indicates if the BZBS authentication token exists.
* `hasJwtToken`: Indicates if the JWT token exists.
* `hasEWalletToken`: Indicates if the e-wallet token exists.
* `hasWalletToken`: Indicates if the wallet token exists.
* `hasFcmToken`: Indicates if the FCM token exists.
* `authTokenLength`: Length of the BZBS token, or 0 if null.
* `jwtTokenLength`: Length of the JWT token, or 0 if null.
* `eWalletTokenLength`: Length of the e-wallet token, or 0 if null.
* `walletTokenLength`: Length of the wallet token, or 0 if null.
* `fcmTokenLength`: Length of the FCM token, or 0 if null.

## Locale Management

The AuthProvider also manages user locale preferences with the following locale codes:

* `1054`: Thai locale
* `1033`: English locale
* Other locale codes as supported by your application

```kotlin
// Save user's preferred locale
auth.saveLocale(1054) // Set to Thai

// Retrieve current locale
val currentLocale = auth.getLocale()
when (currentLocale) {
    1054 -> println("Current locale: Thai")
    1033 -> println("Current locale: English")
    null -> println("No locale preference set")
    else -> println("Locale: $currentLocale")
}

// Check if locale is set
if (auth.hasLocale()) {
    println("User has locale preference")
}

// Remove locale preference
auth.removeLocale()
```

---

End of AuthProvider usage guide.
