# Buzzebees SDK Image Loading Guide

This guide shows how to load Buzzebees images with automatic authentication headers using popular image loading libraries.

## Quick Start

The Buzzebees SDK provides `BuzzebeesSDKImageHeaders` class to automatically add authentication headers (`App-Id` and `Ocp-Apim-Subscription-Key`) to image requests.

```kotlin
import com.buzzebees.sdk.image.BuzzebeesSDKImageHeaders
```

## Setup Examples

### 1. Coil (Recommended)

**Add dependency:**
```kotlin
implementation "io.coil-kt:coil:2.5.0"
```

**Setup in Application class:**
```kotlin
class MyApplication : Application(), ImageLoaderFactory {
    
    override fun onCreate() {
        super.onCreate()
        BuzzebeesSDK.initialize(this)
    }
    
    override fun newImageLoader(): ImageLoader {
        return ImageLoader.Builder(applicationContext)
            .crossfade(true)
            .okHttpClient {
                OkHttpClient.Builder()
                    .addInterceptor(BuzzebeesSDKImageHeaders.createInterceptor())
                    .connectTimeout(30, TimeUnit.SECONDS)
                    .readTimeout(30, TimeUnit.SECONDS)
                    .build()
            }
            .build()
    }
}
```

**Usage:**
```kotlin
val imageView = findViewById<ImageView>(R.id.profile_image)
val profileUrl = BuzzebeesSDK.instance().buildProfileImageUrl("BZD_00001280200")

imageView.load(profileUrl) {
    crossfade(true)
    placeholder(R.drawable.placeholder)
    error(R.drawable.error)
}
```

### 2. Glide

**Add dependencies:**
```kotlin
implementation 'com.github.bumptech.glide:glide:4.16.0'
implementation 'com.github.bumptech.glide:okhttp3-integration:4.16.0'
```

**Create GlideModule:**
```kotlin
@GlideModule
class MyGlideModule : AppGlideModule() {
    
    override fun registerComponents(context: Context, glide: Glide, registry: Registry) {
        val okHttpClient = OkHttpClient.Builder()
            .addInterceptor(BuzzebeesSDKImageHeaders.createInterceptor())
            .connectTimeout(30, TimeUnit.SECONDS)
            .readTimeout(30, TimeUnit.SECONDS)
            .build()
            
        registry.replace(GlideUrl::class.java, InputStream::class.java, OkHttpUrlLoader.Factory(okHttpClient))
    }
    
    override fun applyOptions(context: Context, builder: GlideBuilder) {
        builder.setMemoryCache(LruResourceCache(20 * 1024 * 1024)) // 20MB
    }
}
```

**Usage:**
```kotlin
val imageView = findViewById<ImageView>(R.id.profile_image)
val profileUrl = BuzzebeesSDK.instance().buildProfileImageUrl("BZD_00001280200")

Glide.with(this)
    .load(profileUrl)
    .placeholder(R.drawable.placeholder)
    .error(R.drawable.error)
    .into(imageView)
```

### 3. Picasso

**Add dependency:**
```kotlin
implementation 'com.squareup.picasso:picasso:2.8'
```

**Setup in Application class:**
```kotlin
class MyApplication : Application() {
    
    override fun onCreate() {
        super.onCreate()
        
        BuzzebeesSDK.initialize(this)
        
        val okHttpClient = OkHttpClient.Builder()
            .addInterceptor(BuzzebeesSDKImageHeaders.createInterceptor())
            .connectTimeout(30, TimeUnit.SECONDS)
            .readTimeout(30, TimeUnit.SECONDS)
            .build()
        
        val picasso = Picasso.Builder(this)
            .downloader(OkHttp3Downloader(okHttpClient))
            .build()
            
        Picasso.setSingletonInstance(picasso)
    }
}
```

**Usage:**
```kotlin
val imageView = findViewById<ImageView>(R.id.profile_image)
val profileUrl = BuzzebeesSDK.instance().buildProfileImageUrl("BZD_00001280200")

Picasso.get()
    .load(profileUrl)
    .placeholder(R.drawable.placeholder)
    .error(R.drawable.error)
    .into(imageView)
```

### 4. Without External Library (Manual Loading)

**No additional dependencies needed.**

**Usage:**
```kotlin
val imageView = findViewById<ImageView>(R.id.profile_image)

// Get URL and load with your preferred method
val profileUrl = BuzzebeesSDK.instance().buildProfileImageUrl("BZD_00001280200")
val campaignUrl = BuzzebeesSDK.instance().buildFullImageUrl(null, "campaign_123")

// The interceptor will automatically add headers if you use OkHttpClient
val client = OkHttpClient.Builder()
    .addInterceptor(BuzzebeesSDKImageHeaders.createInterceptor())
    .build()

// Use with your custom image loading solution
```