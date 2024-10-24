---
hide:
  - navigation
  - toc
---

# DNA - vivo Integration Proposals

**Date:** 2024-10-24

Below are the proposed integration steps to accommodate vivo's App Drawer integration product spec.

1. [DNA SDK Integration](#dna-sdk-integration)
2. [App Drawer Recommended Apps Integration](#app-drawer-recommended-apps-integration)
    1. [User opens App Drawer app](#1-user-opens-app-drawer-app)
    2. [Call the DNA SDK to get ads for Slot 4 and Slot 8](#2-call-the-dna-sdk-to-get-ads-for-slot-4-and-slot-8)
    3. [vivo retrieves its own Recommended Apps list](#3-vivo-retrieves-its-own-recommended-apps-list)
    4. [Deduplicate DNA results from vivo's organic results](#4-deduplicate-dna-results-from-vivos-organic-results)
    5. [Load ad creative in the UI](#5-load-ad-creative-in-the-ui)
    6. [Fire impressions for the ads](#6-fire-impressions-for-the-ads)
    7. [Send user click to DNA for routing](#7-send-user-click-to-dna-for-routing)
    8. [Do not cache the results](#8-do-not-cache-the-results)


# DNA SDK Integration
DNA SDK 集成

See [vivo-sdk-changelog](vivo-sdk-changelog.md) for release details.

### 1. Add AAR Dependency
1 添加 AAR 依赖

The DeviceNativeAds SDK is distributed as an AAR file. Follow the instructions below to install it.
DeviceNativeAds SDK 以 AAR 文件的形式分发。请按照以下说明进行安装。

#### 1.1. Download the AAR File
1.1. 下载 AAR 文件

You can find the latest AAR hosted here: [https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.10.aar](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.10.aar)

#### 1.2 Place the AAR File in your Project
1.2 将 AAR 文件放置在您的项目中

Place the DeviceNativeAds SDK in the `libs` folder of your Android project. If you don't have a `libs` folder, create one. It should be placed in the same folder as your `src` folder like so:

将 DeviceNativeAds SDK 放在 Android 项目的 `libs` 文件夹中。如果您没有 `libs` 文件夹，请创建一个。它应该与您的 `src` 文件夹放在同一目录下，如下所示：

```
project-folder/src/main/java/com/example/project/MainActivity.java
project-folder/libs/com.devicenative.dna-vivo-v1.1.10.aar
```

#### 1.3 Add the AAR Dependency
1.3 添加 AAR 依赖

Add the following dependency to your app's `build.gradle` file:

将以下依赖项添加到您应用的 `build.gradle` 文件中：

```gradle
dependencies {
    implementation files('libs/com.devicenative.dna-vivo-v1.1.10.aar')
}
```

or some Gradle versions:

或者对于某些 Gradle 版本：

```gradle
dependencies {
    implementation(files('libs/com.devicenative.dna-vivo-v1.1.10.aar'))
}
```

### 2. Register the Data Orchestrator Service
2 注册数据编排服务


In your AndroidManifest.xml, register the DNADataOrchestrator service and the DNAConfigBuilder service. The DNADataOrchestrator is the main service which coordinates data fetching and processing to deliver fresh advertising results. It will run in your application's process and persist. The DNAConfigBuilder is a one-time use service which runs in a separate process to retrieve the user agent for the device. It will run for approximately 1 second at startup, and not again.

在您的 AndroidManifest.xml 中，注册 DNADataOrchestrator 服务和 DNAConfigBuilder 服务。DNADataOrchestrator 是主要服务，协调数据获取和处理以交付新鲜广告结果。它将在您的应用程序进程中运行并持续存在。DNAConfigBuilder 是一次性服务，在单独的进程中运行以检索设备的 user agent。它将在启动时运行约 1 秒钟，不再运行。


```xml
<service android:name="com.devicenative.dna.DNADataOrchestrator" />
<service
    android:name="com.devicenative.dna.utils.DNAConfigBuilder"
    android:process=":dna_config_builder"
    android:exported="false"/>
```

### 3. Verify Required Permissions
3 验证所需权限

Make sure to include the following permissions in your AndroidManifest.xml:

请确保在您的 AndroidManifest.xml 中包含以下权限：

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.QUERY_ALL_PACKAGES"/>
<uses-permission android:name="android.permission.PACKAGE_USAGE_STATS"/>
```

### 4. Initialize DNA SDK
4 初始化 DNA SDK

Initialize the SDK in your Application class's `onCreate` method:

在您的 Application 类的 `onCreate` 方法中初始化 SDK：

```java
@Override
public void onCreate() {
    super.onCreate();

    DeviceNativeAds dna = DeviceNativeAds.getInstance(this);
    dna.init("4d37ce02-c110-4b06-ad97-9241e4163dd5");

    // any other code you have
}
```

### 7. Clean Up DNA Resources
7. 清理 DNA 资源

In the Application class's `onTerminate` method, clean up SDK resources:

在 Application 类的 `onTerminate` 方法中，清理 SDK 资源：

```java
@Override
public void onTerminate() {
    super.onTerminate();

    DeviceNativeAds dna = DeviceNativeAds.getInstance(this);
    dna.destroy();

    // any other code you have
}
```

# App Drawer Recommended Apps Integration
App Drawer 推荐应用集成

### Introduction
介绍

This section of the documentation will describe the technical integration to integrate re-engagement ads into Slot 4 and Slot 8 of the App Drawer Recommended Apps section.

本文档部分将描述将重新参与广告集成到 App Drawer 推荐应用部分的 Slot 4 和 Slot 8 的技术集成。

### 1: User opens App Drawer app
1. 用户打开 App Drawer 应用

Adding this step to indicate that the user has opened the App Drawer app, which is the trigger for the following logic.

添加此步骤以表明用户已打开 App Drawer 应用，这是以下逻辑的触发器。

### 2. Call the DNA SDK to get ads for Slot 4 and Slot 8
2 调用DNA SDK获取Slot 4和Slot 8的广告

This method call will return a list of DNAResultItem objects which will be used for Slot 4 and Slot 8. It will return 2 total results which can be used for the 2 available slots.

此方法调用将返回一组 DNAResultItem 对象，用于 Slot 4 和 Slot 8。它将返回可用于 2 个可用槽位的 2 个总结果。

```java
DeviceNativeAds dna = DeviceNativeAds.getInstance(getApplicationContext());
List<DNAResultItem> adUnits = dna.getAdsForCache(2, "app drawer, slot 4 and 8");
```

#### Note
注意

1. These are ordered by relevance and monetization potential, so the first ad will be the most relevant.
    - 这些是根据相关性和货币化潜力排序的，所以第一个广告是最相关的。
2. This method return an ad in milliseconds, so it's safe to run on the main thread. 
    - 此方法在毫秒内返回广告，因此可以在主线程上运行。
3. This method does NOT assume that you will show the ads, and therefore will not fire impressions. You must fire impressions yourself with a separate call.
    - 此方法不假设您会显示广告，因此不会触发展示。您必须使用单独的调用触发展示。

#### Key Fields of DNAResultItem Class
DNAResultItem 类的关键字段

| Parameter | Description | Translation |
|----------|-------------|-------------|
| `id` | Unique identifier for the ad. Just a UUID for reference if you need | 广告的唯一标识符。如果需要,只是一个用于参考的UUID |
| `packageName` | The package name of the advertiser's app | 广告主应用的包名 |
| `isInstalled` | A convenient boolean indicating whether the advertiser's app is installed, derived from package manager | 一个方便的布尔值,表示广告主的应用是否已安装,从包管理器派生 |
| `appName` | The name of the advertiser's app | 广告主应用的名称 |
| `className` | The class name of the activity to be shown to the user. Can be null! | 要向用户显示的活动类名。可以为空！ |
| `title` | The ad creative title to be shown to the user | 要向用户显示的广告创意标题 |
| `description` | The ad creative description to be shown to the user. Can be null! | 要向用户显示的广告创意描述。可以为空！ |
| `iconUrl` | The ad creative icon URL to be shown to the user. Can be null! | 要向用户显示的广告创意图标URL。可以为空！ |
| `ratings` | The number of ratings of the advertiser's app from Google Play | 广告主应用从Google Play的评分次数 |
| `downloads` | The number of downloads of the advertiser's app from Google Play | 广告主应用从Google Play的下载次数 |
| `rating` | The average rating of the advertiser's app from Google Play | 广告主应用从Google Play的平均评分 |
| `eCPM` | The expected revenue per thousand impressions for the ad unit. Note that this is not real when the `learningMode` is true | 广告单元的预期每千次展示收入。注意，当 `learningMode` 为 true 时，这不是真实的 |
| `learningMode` | A boolean indicating whether the ad unit is in eCPM learning mode, and whether the eCPM number can be used. | 一个布尔值，表示广告单元是否处于 eCPM 学习模式，以及是否可以使用 eCPM 数字。 |

### 3. vivo retrieves its own Recommended Apps list
3 vivo检索自己的推荐应用列表

In this step, vivo will retrieve its own list of recommended apps. This is the code that currently powers the "Recommended Apps" section of the App Drawer app.

在此步骤中，vivo将检索其自己的推荐应用列表。这是当前为 App Drawer 应用的“推荐应用”部分提供动力的代码。

### 4. Deduplicate DNA results from vivo's organic results
4 为3种场景拆分DNA结果

vivo should prioritize the DNA results over its own organic results, and should therefore should find and remove all of the duplicate package names from their own results.

vivo 应该优先考虑 DNA 结果而不是其自己的有机结果，因此应该找到并删除其结果中的所有重复包名。

```
function removeDuplicateResults(vivoResults, dnaResults):
    # Create a set to store package names from DNA results for quick lookup
    dnaPackageNames = set()

    # Populate the set with package names from DNA results
    for dnaResult in dnaResults:
        dnaPackageNames.add(dnaResult.packageName)

    # Create a new list to store non-duplicate vivo results
    filteredVivoResults = []

    # Iterate over vivo's results
    for vivoResult in vivoResults:
        # Check if the package name is not in the DNA package names set
        if vivoResult.packageName not in dnaPackageNames:
            # If not a duplicate, add to the filtered list
            filteredVivoResults.append(vivoResult)

    # Return the filtered list of vivo results
    return filteredVivoResults 
```

If there are fewer than 2 duplicates, then vivo should remove the lowest relevant results to make room for the 2 DNA results.

如果少于 2 个重复项，则 vivo 应该删除最低相关的结果以腾出空间给 2 个 DNA 结果。

### 5. Load ad creative in the UI
5 在UI中加载广告创意

Below shows an example implementation of loading the ad creative in the UI, with DNA method calls.

以下显示了在 UI 中加载广告创意的示例实现，使用 DNA 方法调用。

```java
ImageView itemIcon = itemView.findViewById(R.id.item_icon);
TextView itemTitle = itemView.findViewById(R.id.item_title);
TextView itemDescription = itemView.findViewById(R.id.item_description);

// handle loading app icon async if app uninstalled
// 如果应用未安装，异步加载应用图标
if (!resultItem.isInstalled) {
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
              Log.e("SearchActivity", "Error loading app icon: " + e.getMessage());
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
        Log.e("SearchActivity", "Error loading app icon: " + e.getMessage());
      }
    }
  });

  // Note, if convenient, there is a sync method to get the icon but not recommended for UI thread
  // Drawable icon = resultItem.loadCreativeDrawable();
  // 注意，如果方便的话，有一个同步方法可以获取图标，但不建议在UI线程中使用
  // Drawable icon = resultItem.loadCreativeDrawable();
} else {
  // if app is installed, show the app icon
  // 如果应用已安装，显示应用图标
  Drawable icon = getPackageManager().getApplicationIcon(resultItem.packageName);
  itemIcon.setImageDrawable(icon);
}

itemTitle.setText(resultItem.title);
```

### 6. Fire impressions for the ads
6 使用新的位置标签为广告触发展示

It is important that you fire impressions for the ads when they are shown to the user. This is how DNA tracks the performance of the ads, but also manages frequency caps, targeting and many other functions.

当向用户显示广告时，重要的是触发展示。这是 DNA 跟踪广告性能的方式，但也管理频率上限、定位和许多其他功能。

```java
DeviceNativeAds dna = DeviceNativeAds.getInstance(getApplicationContext());
dna.fireImpressions(dnaResultsShown);
```

### 7. Send user click to DNA for routing
7 将用户点击发送给DNA进行路由

After the user clicks on a DNA result, vivo will send the click to DNA for routing. DNA should handle the click routing because it is important to deep link the user to the advertiser's app with the appropriate parameters.

用户点击 DNA 结果后，vivo 将点击发送给 DNA 进行路由。DNA 应该处理点击路由，因为重要的是使用适当的参数将用户深度链接到广告主的应用。

```java
DeviceNativeAds dna = DeviceNativeAds.getInstance(getApplicationContext());
DeviceNativeAds.getInstance(this).fireClickAndRoute(resultItem,  new DeviceNativeClickHandler() {
  @Override
  public void onClickServerCompleted() {
    Log.i("GlobalSearchActivity", "Click tracking completed successfully.");
  }

  @Override
  public void onClickRouterCompleted(boolean didRoute) {
    if (!didRoute) {
      Log.e("GlobalSearchActivity", "Error routing click: No activity found to handle the click.");
    } else {
      Log.i("GlobalSearchActivity", "Click routed successfully.");
    }
  }

  @Override
  public void onFailure(int errorCode, String errorMessage) {
    Log.e("GlobalSearchActivity", "Error clicking ad: " + errorMessage);
  }
});
```

#### Important click handling notes
重要的点击处理注意事项

1. Make sure to handle the failure cases, such as no activity found to handle the click, and errors in the click tracking and routing. This should be very rare, but it's important to handle it gracefully.
    - 确保处理故障情况，例如没有活动可以处理点击，以及点击跟踪和路由中的错误。这种情况应该很少见，但重要的是要优雅地处理它。

### 8. Do not cache the results
8 不要缓存结果

Fresh ad retrieval is very low latency and not resource intensive, so there is no need to cache the results. Results can be become stale very quickly, and it's important that the user sees the most relevant results for optimum CPM.

新鲜的广告检索延迟非常低，不占用资源，因此无需缓存结果。结果可能会很快变得过时，用户看到最相关的结果对于最佳 CPM 至关重要。
