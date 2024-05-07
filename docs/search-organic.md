---
hide:
  - footer
---
# Search Integration Steps

The instructions below will guide you on how to integrate the DNA SDK to power ads-only experiences in your launcher app.

## 1. Retrieve Results for Search Results

To retrieve suggested apps for immediate display, use the following code. 

```java
DeviceNativeAds dna = DeviceNativeAds.getInstance(getApplicationContext());
List<DNAResultItem> searchResults = dna.getOrganicResultsForSearch(query);
```

*Note*:

- There is no need to debounce this call, as we have some simple logic to handle that.
- Results will be ranked by relevance, so the first result will be the most relevant.
- Ads will be mixed into the link results
- This method return an results in milliseconds, so it's safe to run on the main thread. 
- This will automatically fire an impression immediately if the impressionUrl is populated for each ad. You MUST therefore show all of the ads to the user.

### Key Fields of DNAResultItem Class

- `id`: Unique identifier for the ad. Just a UUID for reference if you need
- `resultType` : The type of result. Will be `DNAResultItem.TYPE_AD` for ads.
    - `DNAResultItem.TYPE_APP`: For organic app results,
    - `DNAResultItem.TYPE_AD`: For advertisements
    - `DNAResultItem.TYPE_SHORTCUT`: For shortcuts and deep links (only returned for getOrganicLinkSuggestions)
    - `DNAResultItem.TYPE_NOTIFICATION`: For notificiations (only returned for getOrganicLinkSuggestions)
- `packageName`: The package name of the advertiser's app
- `isInstalled`: A convenient boolean indicating whether the advertiser's app is installed, derived from package manager
- `appName`: The name of the advertiser's app
- `title`: The ad creative title to be shown to the user
- `description`: The ad creative description to be shown to the user. Can be null!
- `iconUrl`: The ad creative icon URL to be shown to the user. Can be null!
- `clickUrl`: The click URL of the ad unit. This will automatically be fired by the SDK when using the click and route method.
- `impressionUrl`: The impression URL of the ad unit. This will automatically be fired by the SDK when requesting an ad for display.

### Example Implementation

Here's an example iteration through the deep link results to show an example implementation:

```java
List<DNAResultItem> results = DeviceNativeAds.getInstance(this).getOrganicLinkSuggestions(3);
for (DNAResultItem resultItem : results) {
    View itemView = getLayoutInflater().inflate(R.layout.result_view, section, false);

    ImageView itemIcon = itemView.findViewById(R.id.item_icon);
    TextView itemTitle = itemView.findViewById(R.id.item_title);
    TextView itemDescription = itemView.findViewById(R.id.item_description);

    // Please indent the link results for best visual appearance
    if (!resultItem.resultType.equals(DNAResultItem.TYPE_APP) && !resultItem.resultType.equals(DNAResultItem.TYPE_AD)) {
        insertLeftPadding(itemIcon, 16);
    }

    // Load the icon in the case where it is not just the app icon
    if (resultItem.resultType.equals(DNAResultItem.TYPE_AD) || resultItem.resultType.equals(DNAResultItem.TYPE_SHORTCUT)) {
        resultItem.loadCreativeDrawableAsync(this, new DNAResultItem.ImageCallback() {
            @Override
            public void onImageLoaded(Drawable icon) {
                new Thread(() -> {
                    runOnUiThread(() -> {
                        if (icon == null) {
                            try {
                                Drawable backupIcon = getPackageManager().getApplicationIcon(resultItem.packageName);
                                itemIcon.setImageDrawable(backupIcon);
                            } catch (Exception e) {
                                Log.e("YourActivity", "Error loading app icon: " + e.getMessage());
                            }
                        } else {
                            itemIcon.setImageDrawable(icon);
                        }
                    });
                }).start();
            }

            @Override
            public void onError(String message) {
                try {
                    Drawable icon = getPackageManager().getApplicationIcon(resultItem.packageName);
                    itemIcon.setImageDrawable(icon);
                } catch (Exception e) {
                    Log.e("YourActivity", "Error loading app icon: " + e.getMessage());
                }
            }
        });
    } else {
        try {
            Drawable icon = getPackageManager().getApplicationIcon(resultItem.packageName);
            itemIcon.setImageDrawable(icon);
        } catch (Exception e) {
            Log.e("YourActivity", "Error loading app icon: " + e.getMessage());
        }
    }

    itemTitle.setText(resultItem.title);
    String description = resultItem.description;
    if (resultItem.resultType.equals(DNAResultItem.TYPE_AD)) {
        if (description != null && !description.isEmpty()) {
            description = description.concat(" - Ad");
        } else {
            description = "Promoted";
        }
    }
    if (description == null || description.isEmpty()) {
        itemDescription.setVisibility(View.GONE);
    } else {
        itemDescription.setText(description);
    }

    itemView.setOnClickListener(view -> {
        // Show some loading bar while the click is being processed
        DeviceNativeAds.getInstance(this).fireClickAndRoute(resultItem, new DeviceNativeClickHandler() {
            @Override
            public void onAdClickRouterCompleted(boolean didRoute) {
                // stop showing a loading bar, or handle routing yourself if didRoute is false
            }

            @Override
            public void onFailure(int errorCode, String errorMessage) {
                // Log the error
            }
        });
    });

    linksSection.addView(itemView);
}
```

Here's an example implementation that inserts the left padding.

```java
private void insertLeftPadding(View view, int padding) {
    // Convert dp to pixels
    final float scale = getResources().getDisplayMetrics().density;
    int leftPaddingInPx = (int) (padding * scale + 0.5f);

    // Get existing paddings for top, right, and bottom to preserve them.
    int topPadding = view.getPaddingTop();
    int rightPadding = view.getPaddingRight();
    int bottomPadding = view.getPaddingBottom();

    // Set the new padding. Only the left padding is changed.
    view.setPadding(leftPaddingInPx, topPadding, rightPadding, bottomPadding);
}
```

## 2. Loading The Icons

### Case when isInstalled is true

When the app is installed, we recommend just retrieving the icon from the package manager for speed and simplicity.

```java
if (resultItem.isInstalled) {
    // load the icon from the package manager
    Drawable icon = getPackageManager().getApplicationIcon(resultItem.packageName);
}
```

### Case when isInstalled is false - load the icon from the iconUrl

When the app is not installed, we have provided a convenient method to load the icon from the iconUrl. You can also retrieve the iconUrl from the ad unit object and handle this yourself if you prefer.

#### Synchronously

To be called on a background thread.

```java
if (!resultItem.isInstalled) {
    // load the icon from the iconUrl
    Drawable icon = resultItem.loadCreativeDrawable();
    // set the image on your UI
    imageView.setImageDrawable(icon);
}
```

#### Asynchrously

Can be called on the main thread.

```java
if (!resultItem.isInstalled) {
    // load the icon from the iconUrl
    resultItem.loadCreativeDrawableAsync(new ImageCallback() {
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
