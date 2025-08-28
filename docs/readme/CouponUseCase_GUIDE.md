## CouponUseCase Guide

This guide shows how to initialize and use every public method in `CouponUseCase`, with suspend
and callback examples where available. The CouponUseCase provides comprehensive coupon code
processing functionality for validating and redeeming promotional codes with detailed status
tracking and points management.

### Getting an instance

```kotlin
val couponService = BuzzebeesSDK.instance().couponUseCase
```

---

### processCoupon

Processes and validates coupon codes, returning detailed status information and points earned.

- Request (caller-supplied)

| Field Name | Description                            | Mandatory | Data Type |
|------------|----------------------------------------|-----------|-----------|
| codes      | Comma-separated string of coupon codes | M         | String    |

- Response (`ProcessCouponResponse`) [source](../buzzebees_sdk/src/main/java/com/buzzebees/sdk/entity/coupon/ProcessCouponResponse.kt)  
  HTTP status: 200

### ProcessCouponResponse Entity Fields

| Field Name | Description                   | Data Type          | JSON Field |
|------------|-------------------------------|--------------------|------------|
| success    | Processing success indicator  | Boolean?           | success    |
| data       | Coupon processing result data | ProcessCouponData? | data       |

### ProcessCouponData Entity Fields

| Field Name | Description                          | Data Type                | JSON Field  |
|------------|--------------------------------------|--------------------------|-------------|
| codes      | Map of codes to their status results | Map<String, CodeStatus>? | codes       |
| errorCount | Number of invalid codes processed    | Int?                     | error_count |
| banUntil   | Ban expiration timestamp if banned   | Any?                     | ban_until   |
| buzzebees  | Buzzebees trace information          | Buzzebees?               | buzzebees   |
| today      | Daily processing count               | Int?                     | today       |

### CodeStatus Entity Fields

| Field Name | Description                    | Data Type | JSON Field  |
|------------|--------------------------------|-----------|-------------|
| status     | Code validation status         | String?   | status      |
| points     | Total points value of the code | Int?      | points      |
| pointsEarn | Points earned from this code   | Int?      | points_earn |

- Usage

```kotlin
// Process single coupon code
val singleCode = "PROMO123"
val result = couponService.processCoupon(singleCode)

// Process multiple coupon codes
val multipleCodes = "SAVE10,WELCOME20,BONUS50"

// Suspend
val result = couponService.processCoupon(multipleCodes)

// Callback
couponService.processCoupon(multipleCodes) { result ->
    when (result) {
        is CouponResult.SuccessCoupon -> {
            // Handle successful coupon processing
            val response = result.result
            val success = response.success
            val data = response.data

            if (success == true && data != null) {
                val codeResults = data.codes
                val errorCount = data.errorCount
                val dailyCount = data.today
                val banUntil = data.banUntil
                val traces = data.buzzebees?.traces

                println("Coupon processing completed successfully!")
                println("Error count: $errorCount")
                println("Daily processing count: $dailyCount")

                // Process individual code results
                codeResults?.forEach { (code, status) ->
                    val codeStatus = status.status
                    val totalPoints = status.points
                    val earnedPoints = status.pointsEarn

                    println("Code: $code")
                    println("Status: $codeStatus")
                    println("Total Points: $totalPoints")
                    println("Earned Points: $earnedPoints")

                    when (codeStatus) {
                        "valid" -> {
                            println("✅ Code $code is valid and redeemed")
                            println("   Points earned: $earnedPoints")
                        }
                        "invalid" -> {
                            println("❌ Code $code is invalid")
                        }
                        "expired" -> {
                            println("⏰ Code $code has expired")
                        }
                        "used" -> {
                            println("🔄 Code $code has already been used")
                        }
                        "limit_exceeded" -> {
                            println("🚫 Code $code usage limit exceeded")
                        }
                    }
                }

                // Check for processing ban
                banUntil?.let { banTime ->
                    println("⚠️ Processing temporarily banned until: $banTime")
                }

                // Display trace information if available
                traces?.forEach { trace ->
                    println("Trace ID: ${trace.id}")
                    println("Points change: ${trace.change}")
                    println("Current balance: ${trace.current}")
                }
            }
        }
        is CouponResult.Error -> {
            // Handle error
            val errorCode = result.error.code
            val errorMessage = result.error.message

            when (errorCode) {
                "INVALID_CODE_FORMAT" -> {
                    // Handle invalid code format
                    println("Invalid coupon code format")
                }
                "TOO_MANY_CODES" -> {
                    // Handle too many codes submitted at once
                    println("Too many codes submitted simultaneously")
                }
                "PROCESSING_BANNED" -> {
                    // Handle temporary processing ban
                    println("Coupon processing temporarily banned")
                }
                "DAILY_LIMIT_EXCEEDED" -> {
                    // Handle daily processing limit
                    println("Daily coupon processing limit exceeded")
                }
                "SERVICE_UNAVAILABLE" -> {
                    // Handle service unavailable
                    println("Coupon service temporarily unavailable")
                }
            }
        }
    }
}
```

### Processing Multiple Codes Examples

```kotlin
// Example 1: Process promotional codes
val promoCodes = "SUMMER2024,WELCOME10,LOYALTY25"
couponService.processCoupon(promoCodes) { result ->
    when (result) {
        is CouponResult.SuccessCoupon -> {
            val data = result.result.data
            data?.codes?.let { codeMap ->
                var totalEarned = 0
                var validCodes = 0

                codeMap.forEach { (code, status) ->
                    if (status.status == "valid") {
                        validCodes++
                        totalEarned += status.pointsEarn ?: 0
                    }
                }

                println("Successfully processed $validCodes valid codes")
                println("Total points earned: $totalEarned")
            }
        }
        is CouponResult.Error -> {
            // Handle error case
        }
    }
}

// Example 2: Handle batch processing with error tracking
val batchCodes = "CODE1,CODE2,INVALID,CODE4,EXPIRED"
couponService.processCoupon(batchCodes) { result ->
    when (result) {
        is CouponResult.SuccessCoupon -> {
            val data = result.result.data
            data?.let { processingData ->
                val totalCodes = processingData.codes?.size ?: 0
                val errorCount = processingData.errorCount ?: 0
                val successCount = totalCodes - errorCount

                println("Batch processing results:")
                println("Total codes: $totalCodes")
                println("Successful: $successCount")
                println("Failed: $errorCount")
                println("Success rate: ${(successCount * 100.0 / totalCodes)}%")

                // Log failed codes
                processingData.codes?.filter {
                    it.value.status != "valid"
                }?.forEach { (code, status) ->
                    println("Failed code $code: ${status.status}")
                }
            }
        }
        is CouponResult.Error -> {
            println("Batch processing failed: ${result.error.message}")
        }
    }
}
```

---

## Summary

The CouponUseCase provides essential coupon code processing functionality for promotional campaigns
and loyalty programs within the Buzzebees SDK. It offers comprehensive validation, points tracking,
and detailed status reporting with support for batch processing and anti-abuse mechanisms including
daily limits and temporary bans.