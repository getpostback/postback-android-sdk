# AppSprint Android SDK

Lightweight mobile attribution SDK for Android. Tracks installs, events, and campaign attribution with offline support.

## Installation

### Gradle (Maven Central)

Add to your app's `build.gradle.kts`:

```kotlin
dependencies {
    implementation("app.appsprint:sdk:1.0.1")
}
```

Make sure `mavenCentral()` is in your repositories:

```kotlin
repositories {
    mavenCentral()
}
```

The SDK already declares `INTERNET` and `com.google.android.gms.permission.AD_ID`, so you do not need to add them to your manifest.

### Manual AAR

Download the AAR from [Releases](https://github.com/getappsprint/appsprint-android-sdk/releases) and add to your project's `libs/` directory:

```kotlin
dependencies {
    implementation(files("libs/appsprint-sdk.aar"))
    implementation("androidx.lifecycle:lifecycle-process:2.8.7")
}
```

## Quick Start

```kotlin
import com.appsprint.sdk.AppSprint
import com.appsprint.sdk.AppSprintConfig
import com.appsprint.sdk.AppSprintEventType

class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()

        val sdk = AppSprint.shared(applicationContext)
        sdk.configure(AppSprintConfig(apiKey = "as_live_your_api_key"))

        sdk.sendEvent(
            AppSprintEventType.PURCHASE,
            name = "premium_upgrade",
            params = mapOf("revenue" to 9.99, "currency" to "USD")
        )

        val attribution = sdk.getAttribution()
        println("Source: ${attribution?.source}")
    }
}
```

## Configuration

```kotlin
val config = AppSprintConfig(
    apiKey = "as_live_...",                  // Required
    apiUrl = "https://api.appsprint.app",    // Default
    enableAppleAdsAttribution = false,       // iOS-only, ignored on Android
    customerUserId = null,                   // Optional: your internal user ID
    autoTrackSessions = true,                // Default: fires session_start on
                                             // configure() and foreground,
                                             // debounced to 30 min.
    autoRefreshAttribution = true,           // Default: refreshes attribution
                                             // on configure() and foreground.
    isDebug = false,                         // Default
    logLevel = 2                             // 0=debug, 1=info, 2=warn, 3=error
)
```

## API Reference

```kotlin
val sdk = AppSprint.shared(context)

// Lifecycle
sdk.configure(config: AppSprintConfig)
sdk.destroy()
sdk.isInitialized(): Boolean

// Events
sdk.sendEvent(type: AppSprintEventType, name: String?, params: Map?)
sdk.sendTestEvent(): TestEventResult
sdk.flush()

// Attribution
sdk.getAttribution(): AttributionResult?
sdk.getAttributionParams(): Map<String, String>
sdk.getAppSprintId(): String?
sdk.refreshAttribution(): AttributionResult?
sdk.enableAppleAdsAttribution(): Boolean    // returns false on Android

// User
sdk.setCustomerUserId(userId: String)

// State
sdk.isSdkDisabled(): Boolean
sdk.clearData()
```

### Event Types

`SESSION_START`, `LOGIN`, `SIGN_UP`, `REGISTER`, `PURCHASE`, `SUBSCRIBE`, `START_TRIAL`, `ADD_PAYMENT_INFO`, `ADD_TO_CART`, `ADD_TO_WISHLIST`, `INITIATE_CHECKOUT`, `VIEW_CONTENT`, `VIEW_ITEM`, `SEARCH`, `SHARE`, `TUTORIAL_COMPLETE`, `ACHIEVE_LEVEL`, `LEVEL_START`, `LEVEL_COMPLETE`, `CUSTOM`

## Privacy

The SDK reads the Google Advertising ID during install registration only, off the main thread. It honors Limit Ad Tracking and drops the all-zero advertising ID, so a bogus value never reaches the backend. Play Install Referrer is collected automatically for Play Store installs.

If you ship this SDK in a published app, include these in your Play Console Data safety answers:

- Advertising ID (collected, used for app functionality and advertising or marketing)
- Device or other IDs (the SDK's internal install identifier and Android's identifier-for-vendors equivalent)
- App activity (event names, parameters, revenue, currency)
- User ID, if you call `setCustomerUserId()`

If your app cannot collect advertising IDs (children's apps, certain regional policies), remove the permission in your host app manifest:

```xml
<manifest xmlns:tools="http://schemas.android.com/tools" ...>
    <uses-permission
        android:name="com.google.android.gms.permission.AD_ID"
        tools:node="remove" />
</manifest>
```

Don't pass raw user PII (email, phone, full name) through `params` or `customerUserId`. Both persist to `SharedPreferences` for retry durability. Use hashed or opaque identifiers (SHA-256 of an email, RevenueCat or Superwall `app_user_id`, your internal user UUID).

## Requirements

- Android API 21+ (Android 5.0)
- Kotlin 1.9+ or Java 17

## License

MIT License. See [LICENSE](LICENSE) for details.
