---
hide:
  - footer
---
# Search Ads Integration Steps

The instructions below will guide you on how to integrate the DNA SDK to power ads-only experiences in your launcher app.

## 1. Retrieve Advertisements for Search

To retrieve an advertisement for search, use the following code.

```java
DeviceNativeAds dna = DeviceNativeAds.getInstance(getApplicationContext());
List<DNAResultItem> adUnits = dna.getAdsForSearch(query);
```

*Note*:

- There is no need to debounce this call, as we have some simple logic to handle that.
- This method return an ad in milliseconds, so it's safe to run on the main thread. 
- This will automatically fire an impression immediately if the impressionUrl is populated for each ad. You MUST therefore show all of the ads to the user.

## 2. Placing Result Items in Search

We recommend that you place the ad units for apps that are currently installed in the first position of the search result list. You may even decide to remove your organic results for the same package names from your list, so there are no duplicate results.

For the ad units for apps that are not installed, we recommend that you place the ad units for apps that are not installed in the last position of your search result list.

```java
for (DNAResultItem adUnit : adUnits) {
    if (adUnit.isInstalled) {
        // place the ad unit in the first position of the search result list
        // maybe remove the organic results for the same package names (adUnit.packageName)
    } else {
        // place the ad unit in the last position of the search result list
    }
}
```

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

## 3. Loading The Advertiser's Icon

### Case when isInstalled is true

When the app is installed, we recommend just retrieving the icon from the package manager for speed and simplicity.

```java
if (adUnit.isInstalled) {
    // load the icon from the package manager
    Drawable icon = getPackageManager().getApplicationIcon(adUnit.packageName);
}
```

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

## 4. Handle User Click Interaction

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
