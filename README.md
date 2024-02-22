# DeviceNativeAds SDK Integration Guide

## Introduction

This guide provides detailed instructions for integrating the DeviceNativeAds SDK into your Android application. The DeviceNativeAds SDK allows for efficient and effective ad serving based on device data.

## Prerequisites

- Android Studio and an Android project.
- Basic knowledge of Android app development and Java.

## Integration Steps

### 1. Create an Account and Get a Device Key

First, create an account at [DeviceNative](https://app.devicenative.com). After registration, obtain your unique device key from the [Settings Page](https://app.devicenative.com/settings), which you'll use in your application to initialize the SDK.

### 2. Add Gradle Dependency

Add the following dependency to your app's `build.gradle` file:

```gradle
dependencies {
    implementation 'com.devicenative:dna:latest-version'
}
```

Replace `latest-version` with the current version of the SDK.

### 3. Register the Data Orchestrator Service

In your AndroidManifest.xml, register the DNADataOrchestrator service:

```xml
<service android:name="com.devicenative.dna.DNADataOrchestrator" />
```

### 4. Add Required Permissions

Make sure to include the following permissions in your AndroidManifest.xml:

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.QUERY_ALL_PACKAGES"/>
<uses-permission android:name="android.permission.PACKAGE_USAGE_STATS"/>
```

### 5. Notification Listener Service (Optional but Recommended)

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

### 6. Initialize the SDK

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

### 7. Clean Up Resources

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

### 8. Process Notifications (Optional but Recommended) 

Device Native can use the recent notification for an app as its creative, creating perosnalized experiences that drive high conversions. It's strongly recommended that you add the notification listener.

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

### 9. Retrieve Advertisements for Recommendations (not Search)

To retrieve an advertisement for immediate display, use the following code. 

```java
DeviceNativeAds dna = DeviceNativeAds.getInstance(getApplicationContext());
List<DNAdUnit> adUnits = dna.getAdsForDisplay(1);
```

*Note*:
- That you can adjust the number of ads to be returned by passing a number to the getAdsForDisplay method.
- This method return an ad in milliseconds, so it's safe to run on the main thread. 
- This will automatically fire an impression immediately if the impressionUrl is populated for each ad. You MUST therefore show all of the ads to the user.
- These are ordered by relevance, so the first ad will be the most relevant.

### 10. Retrieve Advertisements for Search

To retrieve an advertisement for search, use the following code.

```java
DeviceNativeAds dna = DeviceNativeAds.getInstance(getApplicationContext());
List<DNAdUnit> adUnits = dna.getAdsForSearch(query);
```

*Note*:
- There is no need to debounce this call, as we have some simple logic to handle that.
- This method return an ad in milliseconds, so it's safe to run on the main thread. 
- This will automatically fire an impression immediately if the impressionUrl is populated for each ad. You MUST therefore show all of the ads to the user.

#### Dealing with AdUnits in Search

We recommend that you place the ad units for apps that are currently installed in the first position of the search result list. You may even decide to remove your organic results for the same package names from your list, so there are no duplicate results.

For the ad units for apps that are not installed, we recommend that you place the ad units for apps that are not installed in the last position of your search result list.

```java
for (DNAdUnit adUnit : adUnits) {
    if (adUnit.isAppInstalled) {
        // place the ad unit in the first position of the search result list
        // maybe remove the organic results for the same package names (adUnit.packageName)
    } else {
        // place the ad unit in the last position of the search result list
    }
}
```

#### Key Fields of DNAdUnit Class

- `adId`: Unique identifier for the ad. Just a UUID for reference if you need
- `packageName`: The package name of the advertiser's app
- `isAppInstalled`: A convenient boolean indicating whether the advertiser's app is installed, derived from package manager
- `appName`: The name of the advertiser's app
- `title`: The ad creative title to be shown to the user
- `description`: The ad creative description to be shown to the user. Can be null!
- `iconUrl`: The ad creative icon URL to be shown to the user. Can be null!
- `clickUrl`: The click URL of the ad unit. This will automatically be fired by the SDK when using the click and route method.
- `impressionUrl`: The impression URL of the ad unit. This will automatically be fired by the SDK when requesting an ad for display.

#### Loading the advertiser's icon

##### Case when isAppInstalled is true

When the app is installed, we recommend just retrieving the icon from the package manager for speed and simplicity.

```java
if (adUnit.isAppInstalled) {
    // load the icon from the package manager
    Drawable icon = getPackageManager().getApplicationIcon(adUnit.packageName);
}
```

##### Case when isAppInstalled is false - load the icon from the iconUrl

When the app is not installed, we have provided a convenient method to load the icon from the iconUrl. You can also retrieve the iconUrl from the ad unit object and handle this yourself if you prefer.

```java
if (!adUnit.isAppInstalled) {
    // load the icon from the iconUrl
    adUnit.loadCreativeDrawable(new ImageCallback() {
    @Override
    public void onImageLoaded(Drawable icon) {
        // Run on UI thread if updating UI components
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                // set the image on your UI
                imageView.setImageDrawable(image);
            }
        });
    }

    @Override
    public void onError(String error) {
        // Log the error, show a default icon, etc
    }
});
}
```

### 11. Handle User Click Interaction

When a user clicks on the ad, use the following code to handle the routing and receive notifications of status.

It executes on a separate thread to ensure the click handling URL properly tracks before the user is sent to the destination, and loading could take a second, so it's recommended to show a loading indicator until the callback is fired. Fine to pass null to the clickHandler callback if you don't need to.

```java
DeviceNativeAds dna = DeviceNativeAds.getInstance(getApplicationContext());
dna.fireClickAndRoute(adUnit, new DeviceNativeAdClickHandler() {
    /**
     * This method is called when the ad click routing process is completed, which means the user was
     * sent to their destination, or it failed to route for soem reason.
     * @param didRoute A boolean indicating whether the routing was successful.
     */
    public void onAdClickRouterCompleted(boolean didRoute) {
        // stop showing a loading bar, or handle routing yourself if didRoute is false
    }

    /**
     * This method is called when there is a failure in the ad click process. Implement this method to
     * define what should happen when there is a failure in the ad click process.
     * @param errorCode An integer representing the error code of the failure.
     * @param errorMessage A string representing the error message of the failure.
     */
    public void onFailure(int errorCode, String errorMessage) {
        // log the fail, stop showing loading bar, etc
    }
});
```

## Need Help?

Please email help@devicenative.com for assistance or questions about the process.
