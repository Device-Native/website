---
hide:
  - footer
---
# Basic SDK Integration Steps

These are the required steps to integrate the Device Native SDK into your Android launcher app. The SDK is intended to be integrated with Android launcher apps that have system-level access.

## 1. Create an Account and Get a Device Key

First, create an account at [DeviceNative](https://app.devicenative.com). After registration, obtain your unique device key from the [Settings Page](https://app.devicenative.com/settings), which you'll use in your application to initialize the SDK.

## 2. Add AAR Dependency

The DeviceNativeAds SDK is distributed as an AAR file. Follow the instructions below to install it.

### 2.1. Download the AAR File

You can find the latest AAR hosted here: [https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-v1.0.2.aar](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-v1.0.2.aar)

### 2.2 Place the AAR File in your Project

Place the DeviceNativeAds SDK in the `libs` folder of your Android project. If you don't have a `libs` folder, create one. It should be placed in the same folder as your `src` folder like so:

```
project-folder/src/main/java/com/example/project/MainActivity.java
project-folder/libs/com.devicenative.dna-v1.0.2.aar
```

### 2.3 Add the AAR Dependency

Add the following dependency to your app's `build.gradle` file:

```gradle
dependencies {
    implementation files('libs/com.devicenative.dna-v1.0.2.aar')
}
```

or some Gradle versions:

```gradle
dependencies {
    implementation(files('libs/com.devicenative.dna-v1.0.2.aar'))
}
```

## 3. Register the Data Orchestrator Service

In your AndroidManifest.xml, register the DNADataOrchestrator service:

```xml
<service android:name="com.devicenative.dna.DNADataOrchestrator" />
```

## 4. Add Required Permissions

Make sure to include the following permissions in your AndroidManifest.xml:

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.QUERY_ALL_PACKAGES"/>
<uses-permission android:name="android.permission.PACKAGE_USAGE_STATS"/>
```

## 5. Notification Listener Service (Optional but Recommended)

If you want to collect notifications for ad creative, you must have a Notification Listener registered like the example below.

**No action needed here**, but just remember the name of this class for later.

```xml
<service android:name=".notification.NotificationListener"
         android:enabled="true"
         android:exported="true"
         android:permission="android.permission.BIND_NOTIFICATION_LISTENER_SERVICE">
    <intent-filter>
        <action android:name="android.service.notification.NotificationListenerService" />
    </intent-filter>
</service>
```

## 6. Initialize the SDK

Initialize the SDK in your Application class's `onCreate` method:

```java
@Override
public void onCreate() {
    super.onCreate();

    DeviceNativeAds dna = DeviceNativeAds.getInstance(this);
    dna.init("YOUR_DEVICE_KEY");

    // any other code you have
}
```

Replace `YOUR_DEVICE_KEY` with the key obtained in step 1.

## 7. Clean Up Resources

In the Application class's `onTerminate` method, clean up SDK resources:

```java
@Override
public void onTerminate() {
    super.onTerminate();

    DeviceNativeAds dna = DeviceNativeAds.getInstance(this);
    dna.destroy();

    // any other code you have
}
```

## 8. Process Notifications (Optional but Recommended) 

Device Native can use the recent notification for an app as its creative, creating personalized experiences that drive high conversions. It's strongly recommended that you add the notification listener.

Open your notficiation listener class that you noticed in Step 5:
```java
// This is your class
public class NotificationListener extends NotificationListenerService {
```

Find the listener service method that handles new notifications, and call the appropriate Device Native code as shown below:
```java
@Override
public void onNotificationPosted(StatusBarNotification sbn) {

    DeviceNativeAds dna = DeviceNativeAds.getInstance(getApplicationContext());
    dna.onNotificationPosted(sbn);

    // your other handling code
}
```

## Need Help?

Please email [help@devicenative.com](help@devicenative.com) for assistance or questions about the process.
