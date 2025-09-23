# Custom API Integration Guide

This guide provides comprehensive documentation for creating custom API services using the BuzzebeesSDK's pre-configured HTTP infrastructure. The custom API feature allows developers to leverage the SDK's optimized OkHttp and Retrofit configuration while building their own API integrations.

---

## Table of Contents

1. [Overview](#overview)
2. [Quick Start](#quick-start)
3. [Core Components](#core-components)
4. [Usage Patterns](#usage-patterns)
5. [Advanced Configuration](#advanced-configuration)
6. [Custom Interceptors](#custom-interceptors)
7. [Best Practices](#best-practices)
8. [Examples](#examples)
9. [Migration Guide](#migration-guide)

---

## Overview

The Custom API Integration feature allows you to:

- **Reuse SDK Configuration**: Leverage pre-configured timeouts, authentication, and interceptors
- **Create Custom APIs**: Build your own Retrofit API services with minimal setup
- **Advanced Configuration**: Customize base URLs, add interceptors, and modify converter factories
- **Secure API Access**: Create custom API services safely without exposing internal details
- **Performance Optimization**: Share connection pools and thread pools with the SDK

### Benefits

✅ **Consistent Configuration**: All APIs use the same optimized settings  
✅ **Easy Integration**: Simple methods for basic use cases  
✅ **Advanced Features**: Builder pattern for complex requirements  
✅ **Performance**: Shared resources and connection pooling  
✅ **Authentication**: Automatic inclusion of SDK authentication headers  
✅ **Maintainability**: Centralized HTTP configuration  

---

## Quick Start

### 1. Simple Custom API Service

```kotlin
// Define your API interface
interface WeatherApi {
    @GET("weather/current")
    suspend fun getCurrentWeather(@Query("city") city: String): WeatherResponse
}

// Create the API service using SDK configuration
val weatherApi = BuzzebeesSDK.instance().createCustomApiService(WeatherApi::class.java)

// Use it
val weather = weatherApi.getCurrentWeather("Bangkok")
```

### 2. Custom API with Different Base URL

```kotlin
interface PaymentApi {
    @POST("payments/process")
    suspend fun processPayment(@Body request: PaymentRequest): PaymentResponse
}

val paymentApi = BuzzebeesSDK.instance().customApiBuilder()
    .baseUrl("https://payment-gateway.com/api/v1/")
    .build(PaymentApi::class.java)
```

### 3. Custom API with Additional Authentication

```kotlin
class ApiKeyInterceptor(private val apiKey: String) : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        val request = chain.request().newBuilder()
            .addHeader("X-API-Key", apiKey)
            .build()
        return chain.proceed(request)
    }
}

val secureApi = BuzzebeesSDK.instance().customApiBuilder()
    .baseUrl("https://secure-api.com/v1/")
    .addInterceptor(ApiKeyInterceptor("your-api-key"))
    .build(SecureApi::class.java)
```

---

## Core Components

### BuzzebeesSDK Custom API Methods

| Method | Description | Return Type |
|--------|-------------|-------------|
| `createCustomApiService(Class<T>)` | Create a simple custom API service | `T` |
| `customApiBuilder()` | Get a builder for advanced configuration | `ICustomApiBuilder` |

### ICustomApiBuilder Methods

| Method | Description | Return Type |
|--------|-------------|-------------|
| `baseUrl(String)` | Set custom base URL | `ICustomApiBuilder` |
| `addInterceptor(Interceptor)` | Add additional interceptor | `ICustomApiBuilder` |
| `addInterceptors(Collection<Interceptor>)` | Add multiple interceptors | `ICustomApiBuilder` |
| `converterFactory(Converter.Factory)` | Set custom converter factory | `ICustomApiBuilder` |
| `build(Class<T>)` | Build single API service | `T` |
| `buildMultiple(Class<*>...)` | Build multiple API services | `List<Any>` |

---

## Usage Patterns

### Pattern 1: Service Layer Integration

```kotlin
class WeatherService {
    private val weatherApi = BuzzebeesSDK.instance().createCustomApiService(WeatherApi::class.java)
    
    suspend fun getCurrentWeather(city: String): Result<WeatherResponse> {
        return try {
            val response = weatherApi.getCurrentWeather(city)
            Result.success(response)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
    
    suspend fun getForecast(city: String, days: Int = 5): Result<ForecastResponse> {
        return try {
            val response = weatherApi.getForecast(city, days)
            Result.success(response)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

### Pattern 2: Repository Pattern

```kotlin
interface WeatherRepository {
    suspend fun getCurrentWeather(city: String): Result<Weather>
    suspend fun getForecast(city: String, days: Int): Result<List<DailyForecast>>
}

class WeatherRepositoryImpl : WeatherRepository {
    private val weatherApi = BuzzebeesSDK.instance().customApiBuilder()
        .baseUrl("https://weather-api.com/v2/")
        .addInterceptor(WeatherApiKeyInterceptor())
        .build(WeatherApi::class.java)
    
    override suspend fun getCurrentWeather(city: String): Result<Weather> {
        return try {
            val response = weatherApi.getCurrentWeather(city)
            Result.success(response.toWeather())
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
    
    override suspend fun getForecast(city: String, days: Int): Result<List<DailyForecast>> {
        return try {
            val response = weatherApi.getForecast(city, days)
            Result.success(response.forecast)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

### Pattern 3: File Operations with Custom APIs

```kotlin
// Create a dedicated API interface for file operations
interface FileApi {
    @Multipart
    @POST("upload")
    suspend fun uploadFile(
        @Part file: MultipartBody.Part
    ): UploadResponse
    
    @GET
    suspend fun downloadFile(@Url downloadUrl: String): ResponseBody
}

class FileUploadService {
    private val fileApi = BuzzebeesSDK.instance().customApiBuilder()
        .baseUrl("https://file-service.com/api/")
        .build(FileApi::class.java)
    
    suspend fun uploadFile(file: File): Boolean {
        return try {
            val requestBody = file.asRequestBody("application/octet-stream".toMediaType())
            val filePart = MultipartBody.Part.createFormData("file", file.name, requestBody)
            val response = fileApi.uploadFile(filePart)
            response.success == true
        } catch (e: Exception) {
            false
        }
    }
    
    suspend fun downloadFile(downloadUrl: String): ByteArray? {
        return try {
            val response = fileApi.downloadFile(downloadUrl)
            response.bytes()
        } catch (e: Exception) {
            null
        }
    }
}
```

### Pattern 4: Multiple APIs with Shared Configuration

```kotlin
class UnifiedApiService {
    private val apis = BuzzebeesSDK.instance().customApiBuilder()
        .baseUrl("https://unified-platform.com/api/")
        .addInterceptor(PlatformAuthInterceptor())
        .addInterceptor(CustomLoggingInterceptor("UnifiedAPI"))
        .buildMultiple(
            UserApi::class.java,
            ProductApi::class.java,
            OrderApi::class.java
        )
    
    private val userApi = apis[0] as UserApi
    private val productApi = apis[1] as ProductApi  
    private val orderApi = apis[2] as OrderApi
    
    suspend fun getUserProfile(userId: String) = userApi.getProfile(userId)
    suspend fun getProducts(category: String) = productApi.getByCategory(category)
    suspend fun createOrder(order: OrderRequest) = orderApi.create(order)
}
```

---

## Advanced Configuration

### Custom Base URL and Converter Factory

```kotlin
val customApi = BuzzebeesSDK.instance().customApiBuilder()
    .baseUrl("https://api.example.com/v2/")
    .converterFactory(MoshiConverterFactory.create())
    .build(CustomApi::class.java)
```

### Multiple Interceptors

```kotlin
val enhancedApi = BuzzebeesSDK.instance().customApiBuilder()
    .baseUrl("https://enhanced-api.com/v1/")
    .addInterceptor(AuthenticationInterceptor())
    .addInterceptor(RateLimitInterceptor(maxRequestsPerSecond = 10))
    .addInterceptor(CustomLoggingInterceptor("EnhancedAPI"))
    .addInterceptor(RetryInterceptor(maxRetries = 3))
    .build(EnhancedApi::class.java)
```

### Building Multiple APIs

```kotlin
val (weatherApi, paymentApi, notificationApi) = BuzzebeesSDK.instance().customApiBuilder()
    .baseUrl("https://multi-service.com/api/")
    .addInterceptor(CommonAuthInterceptor())
    .buildMultiple(
        WeatherApi::class.java,
        PaymentApi::class.java,
        NotificationApi::class.java
    )

val weatherService = weatherApi as WeatherApi
val paymentService = paymentApi as PaymentApi
val notificationService = notificationApi as NotificationApi
```

---

## Custom Interceptors

### API Key Authentication

```kotlin
class ApiKeyInterceptor(private val apiKey: String, private val headerName: String = "X-API-Key") : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        val request = chain.request().newBuilder()
            .addHeader(headerName, apiKey)
            .build()
        return chain.proceed(request)
    }
}
```

### Rate Limiting

```kotlin
class RateLimitInterceptor(private val maxRequestsPerSecond: Int = 10) : Interceptor {
    private var lastRequestTime = 0L
    private val minInterval = 1000L / maxRequestsPerSecond

    override fun intercept(chain: Interceptor.Chain): Response {
        val now = System.currentTimeMillis()
        val timeSinceLastRequest = now - lastRequestTime
        
        if (timeSinceLastRequest < minInterval) {
            Thread.sleep(minInterval - timeSinceLastRequest)
        }
        
        lastRequestTime = System.currentTimeMillis()
        return chain.proceed(chain.request())
    }
}
```

### Custom Logging

```kotlin
class CustomLoggingInterceptor(private val tag: String = "CustomAPI") : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        val request = chain.request()
        Log.d(tag, "Request: ${request.method} ${request.url}")
        
        val startTime = System.currentTimeMillis()
        val response = chain.proceed(request)
        val endTime = System.currentTimeMillis()
        
        Log.d(tag, "Response: ${response.code} (${endTime - startTime}ms)")
        return response
    }
}
```

### Retry Interceptor

```kotlin
class RetryInterceptor(private val maxRetries: Int = 3) : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        var request = chain.request()
        var response = chain.proceed(request)
        var retryCount = 0

        while (!response.isSuccessful && retryCount < maxRetries) {
            retryCount++
            response.close()
            response = chain.proceed(request)
        }

        return response
    }
}
```

---

## Best Practices

### 1. Reuse API Instances

Create API services once and reuse them throughout your app:

```kotlin
class ApiManager {
    private val weatherApi by lazy { 
        BuzzebeesSDK.instance().createCustomApiService(WeatherApi::class.java) 
    }
    
    fun getWeatherApi(): WeatherApi = weatherApi
}
```

### 2. Handle Errors Gracefully

Always wrap API calls in try-catch blocks or use Result types:

```kotlin
suspend fun fetchWeather(city: String): Result<WeatherResponse> {
    return try {
        val response = weatherApi.getCurrentWeather(city)
        Result.success(response)
    } catch (e: HttpException) {
        Result.failure(ApiException("HTTP ${e.code()}: ${e.message()}"))
    } catch (e: IOException) {
        Result.failure(NetworkException("Network error: ${e.message}"))
    } catch (e: Exception) {
        Result.failure(UnknownException("Unknown error: ${e.message}"))
    }
}
```

### 3. Use Dependency Injection

Inject API services for better testability:

```kotlin
@Module
class ApiModule {
    @Provides
    @Singleton
    fun provideWeatherApi(): WeatherApi {
        return BuzzebeesSDK.instance().createCustomApiService(WeatherApi::class.java)
    }
    
    @Provides
    @Singleton
    fun providePaymentApi(): PaymentApi {
        return BuzzebeesSDK.instance().customApiBuilder()
            .baseUrl("https://payment-api.com/v1/")
            .addInterceptor(ApiKeyInterceptor(BuildConfig.PAYMENT_API_KEY))
            .build(PaymentApi::class.java)
    }
}
```

### 4. Organize Base URLs

Group related APIs under the same base URL when possible:

```kotlin
object ApiConstants {
    const val WEATHER_BASE_URL = "https://weather-api.com/v2/"
    const val PAYMENT_BASE_URL = "https://payment-api.com/v1/"
    const val NOTIFICATION_BASE_URL = "https://notification-api.com/api/"
}

class ApiServiceFactory {
    fun createWeatherApi(): WeatherApi {
        return BuzzebeesSDK.instance().customApiBuilder()
            .baseUrl(ApiConstants.WEATHER_BASE_URL)
            .build(WeatherApi::class.java)
    }
}
```

### 5. Custom Converter Factories

Use appropriate converter factories for your data format:

```kotlin
// For JSON with Moshi
val moshiApi = BuzzebeesSDK.instance().customApiBuilder()
    .converterFactory(MoshiConverterFactory.create())
    .build(MoshiApi::class.java)

// For XML
val xmlApi = BuzzebeesSDK.instance().customApiBuilder()
    .converterFactory(SimpleXmlConverterFactory.create())
    .build(XmlApi::class.java)

// For Protocol Buffers
val protoApi = BuzzebeesSDK.instance().customApiBuilder()
    .converterFactory(ProtoConverterFactory.create())
    .build(ProtoApi::class.java)
```

---

## Examples

### Real-World Integration Example

```kotlin
// Define file API interface for upload operations
interface FileApi {
    @Multipart
    @POST("products/{productId}/image")
    suspend fun uploadProductImage(
        @Path("productId") productId: String,
        @Part image: MultipartBody.Part
    ): UploadResponse
}

class ECommerceApiService {
    private val productApi = BuzzebeesSDK.instance().customApiBuilder()
        .baseUrl("https://ecommerce-api.com/v2/")
        .addInterceptor(ApiKeyInterceptor(BuildConfig.ECOMMERCE_API_KEY))
        .addInterceptor(CustomLoggingInterceptor("ECommerce"))
        .build(ProductApi::class.java)
    
    private val paymentApi = BuzzebeesSDK.instance().customApiBuilder()
        .baseUrl("https://payment-gateway.com/api/")
        .addInterceptor(PaymentAuthInterceptor(BuildConfig.PAYMENT_SECRET))
        .addInterceptor(RateLimitInterceptor(maxRequestsPerSecond = 5))
        .build(PaymentApi::class.java)
    
    // File operations handled through dedicated API interfaces
    private val fileApi = BuzzebeesSDK.instance().customApiBuilder()
        .baseUrl("https://ecommerce-api.com/v2/")
        .addInterceptor(ApiKeyInterceptor(BuildConfig.ECOMMERCE_API_KEY))
        .build(FileApi::class.java)
    
    suspend fun searchProducts(query: String): List<Product> {
        return try {
            productApi.search(query).products
        } catch (e: Exception) {
            emptyList()
        }
    }
    
    suspend fun processPayment(paymentRequest: PaymentRequest): PaymentResult {
        return try {
            paymentApi.process(paymentRequest)
        } catch (e: HttpException) {
            PaymentResult.failure("Payment failed: ${e.message()}")
        }
    }
    
    suspend fun uploadProductImage(productId: String, imageFile: File): Boolean {
        return try {
            val requestBody = imageFile.asRequestBody("image/jpeg".toMediaType())
            val filePart = MultipartBody.Part.createFormData("image", imageFile.name, requestBody)
            val response = fileApi.uploadProductImage(productId, filePart)
            response.success == true
        } catch (e: Exception) {
            false
        }
    }
}
```

### MVVM Pattern Integration

```kotlin
class ProductViewModel : ViewModel() {
    private val productApi = BuzzebeesSDK.instance().createCustomApiService(ProductApi::class.java)
    
    private val _products = MutableLiveData<List<Product>>()
    val products: LiveData<List<Product>> = _products
    
    private val _loading = MutableLiveData<Boolean>()
    val loading: LiveData<Boolean> = _loading
    
    private val _error = MutableLiveData<String?>()
    val error: LiveData<String?> = _error
    
    fun searchProducts(query: String) {
        viewModelScope.launch {
            _loading.value = true
            _error.value = null
            
            try {
                val response = productApi.search(query)
                _products.value = response.products
            } catch (e: Exception) {
                _error.value = "Failed to search products: ${e.message}"
            } finally {
                _loading.value = false
            }
        }
    }
}
```

---

## Migration Guide

### From Manual OkHttp Configuration

```kotlin
// Before: Manual configuration
val okHttpClient = OkHttpClient.Builder()
    .connectTimeout(30, TimeUnit.SECONDS)
    .readTimeout(30, TimeUnit.SECONDS)
    .addInterceptor(AuthInterceptor())
    .addInterceptor(LoggingInterceptor())
    .build()

val retrofit = Retrofit.Builder()
    .baseUrl("https://api.example.com/")
    .client(okHttpClient)
    .addConverterFactory(GsonConverterFactory.create())
    .build()

val api = retrofit.create(MyApi::class.java)

// After: Using SDK configuration
val api = BuzzebeesSDK.instance().customApiBuilder()
    .baseUrl("https://api.example.com/")
    .addInterceptor(AuthInterceptor())
    .build(MyApi::class.java)
```

### From Retrofit Configuration

```kotlin
// Before: Manual Retrofit setup
class ApiClient {
    private val retrofit = Retrofit.Builder()
        .baseUrl(BASE_URL)
        .addConverterFactory(GsonConverterFactory.create())
        .client(createOkHttpClient())
        .build()
        
    fun <T> createService(serviceClass: Class<T>): T {
        return retrofit.create(serviceClass)
    }
}

// After: Using SDK
class ApiClient {
    fun <T> createService(serviceClass: Class<T>): T {
        return BuzzebeesSDK.instance().createCustomApiService(serviceClass)
    }
    
    fun <T> createServiceWithCustomUrl(serviceClass: Class<T>, baseUrl: String): T {
        return BuzzebeesSDK.instance().customApiBuilder()
            .baseUrl(baseUrl)
            .build(serviceClass)
    }
}
```

### Gradual Migration Strategy

1. **Start with Simple APIs**: Begin by migrating simple API services using `createCustomApiService()`
2. **Add Custom Configuration**: Use `customApiBuilder()` for APIs requiring specific configuration
3. **Update Base URLs**: Gradually update base URLs while maintaining functionality
4. **Use Available Methods**: Use the provided SDK methods for API creation
5. **Test Thoroughly**: Ensure all functionality works with the new configuration
6. **Remove Old Code**: Remove manual HTTP configuration code once migration is complete

---

## Conclusion

The Custom API Integration feature provides a powerful and flexible way to extend your app's API capabilities while leveraging the BuzzebeesSDK's optimized configuration. By following the patterns and best practices outlined in this guide, you can create maintainable, performant, and scalable API integrations that benefit from the SDK's battle-tested HTTP infrastructure.

For additional examples and implementation details, refer to the `/examples/CustomApiExamples.kt` file in the SDK source code.
