---
hide:
  - footer
---
# Hot App Suggestions Integration Steps

The instructions below will guide you on how to integrate the DNA SDK to power hot app recommendations in your launcher app. These are personalized suggestions to go download new apps.

## 1. Retrieve List of Hot Apps

To retrieve the list of hot apps for immediate display, use the following code. 

```java
DeviceNativeAds dna = DeviceNativeAds.getInstance(getApplicationContext());
List<DNAResultItem> adUnits = dna.getHotAppsList(10);
```

*Note*:

- That you can adjust the number of top apps to be returned by passing a number to the getHotAppsList method.
- All of these results will be for apps that are not currently install (`isInstalled` will be false)
- This method return the results in milliseconds, so it's safe to run on the main thread. 
- This will automatically fire an impression immediately if the impressionUrl is populated for each result. You MUST therefore show all of the result to the user.
- These are ordered by relevance, so the first result will be the most relevant.

### Key Fields of DNAResultItem Class

- `id`: Unique identifier for the ad. Just a UUID for reference if you need
- `packageName`: The package name of the advertiser's app
- `isInstalled`: A convenient boolean indicating whether the advertiser's app is installed, derived from package manager
- `appName`: The name of the advertiser's app
- `title`: The ad creative title to be shown to the user
- `description`: The ad creative description to be shown to the user. Can be null!
- `iconUrl`: The ad creative icon URL to be shown to the user. Can be null!
- `clickUrl`: The click URL of the ad unit. This will automatically be fired by the SDK when using the click and route method.
- `impressionUrl`: The impression URL of the ad unit. This will automatically be fired by the SDK when requesting an ad for display.

## 2. Loading The Advertiser's Icon

A reminder again that all of these hot apps results will be for apps that are not currently installed, so you'll need to load the icons asynchronously.

### Case when isInstalled is false - load the icon from the iconUrl

When the app is not installed, we have provided a convenient method to load the icon from the iconUrl. You can also retrieve the iconUrl from the ad unit object and handle this yourself if you prefer.

#### Synchronously

To be called on a background thread.

```java
if (!adUnit.isInstalled) {
    // load the icon from the iconUrl
    Drawable icon = adUnit.loadCreativeDrawable();
    // set the image on your UI
    imageView.setImageDrawable(icon);
}
```

#### Asynchrously

Can be called on the main thread.

```java
if (!adUnit.isInstalled) {
    // load the icon from the iconUrl
    adUnit.loadCreativeDrawableAsync(new ImageCallback() {
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

## 3. Handle User Click Interaction

When a user clicks on the ad, use the following code to handle the routing and receive notifications of status.

It executes on a separate thread to ensure the click handling URL properly tracks before the user is sent to the destination, and loading could take a second, so it's recommended to show a loading indicator until the callback is fired. Fine to pass null to the clickHandler callback if you don't need to.

```java
DeviceNativeAds dna = DeviceNativeAds.getInstance(getApplicationContext());
dna.fireClickAndRoute(adUnit, new DeviceNativeClickHandler() {
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

Please email [help@devicenative.com](help@devicenative.com) for assistance or questions about the process.
