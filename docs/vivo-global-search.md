---
hide:
  - navigation
  - toc
---

# DNA - vivo Integration Proposals

**Date:** 2024-09-09

Below are the proposed integration steps to accommodate vivo's Global Search integration product spec.

1. [DNA SDK Integration](#dna-sdk-integration)
2. [GS Recommended Apps Integration](#gs-recommended-apps-integration)
    1. [User opens Global Search app](#1-user-opens-global-search-app)
    2. [Call the DNA SDK to get ALL recommended apps](#2-call-the-dna-sdk-to-get-all-recommended-apps)
    3. [vivo retrieves its own Recommended Apps list](#3-vivo-retrieves-its-own-recommended-apps-list)
    4. [Split out the DNA results for the 3 scenarios](#4-split-out-the-dna-results-for-the-3-scenarios)
    5. [Load ad creative in the UI](#5-load-ad-creative-in-the-ui)
    6. [Fire impressions for the ads with the new placement tag](#6-fire-impressions-for-the-ads-with-the-new-placement-tag)
    7. [Send user click to DNA for routing](#7-send-user-click-to-dna-for-routing)
3. [GS Search Integration](#gs-search-integration)
    1. [User enters character in search bar](#1-user-enters-character-in-search-bar)
    2. [Call the DNA SDK to get ALL search results](#2-call-the-dna-sdk-to-get-all-search-results)
    3. [Split out the DNA results for the 2 scenarios](#3-split-out-the-dna-results-for-the-2-scenarios)
    4. [Show all scenario 5 results from DNA](#4-show-all-scenario-5-results-from-dna)
    5. [Register impressions for all visible Scenario 5 results](#5-register-impressions-for-all-visible-scenario-5-results)
    6. [Mix scenario 6 results with vivo organic results](#6-mix-scenario-6-results-with-vivo-organic-results)
    7. [Fire impressions for all visible Scenario 6 results](#7-fire-impressions-for-all-visible-scenario-6-results)
    8. [Load ad creative in the UI for either scenario](#8-load-ad-creative-in-the-ui-for-either-scenario)
    9. [Send user click to DNA for routing for either scenario](#9-send-user-click-to-dna-for-routing-for-either-scenario)

以下是适应vivo全局搜索集成产品规格的建议集成步骤。

1. [DNA SDK集成](#dna-sdk-integration)
2. [GS推荐应用集成](#gs-recommended-apps-integration)
    1. [用户打开全局搜索应用](#1-user-opens-global-search-app)
    2. [调用DNA SDK获取所有推荐应用](#2-call-the-dna-sdk-to-get-all-recommended-apps)
    3. [vivo检索自己的推荐应用列表](#3-vivo-retrieves-its-own-recommended-apps-list)
    4. [为3种场景拆分DNA结果](#4-split-out-the-dna-results-for-the-3-scenarios)
    5. [在UI中加载广告创意](#5-load-ad-creative-in-the-ui)
    6. [使用新的位置标签为广告触发展示](#6-fire-impressions-for-the-ads-with-the-new-placement-tag)
    7. [将用户点击发送给DNA进行路由](#7-send-user-click-to-dna-for-routing)
3. [GS搜索集成](#gs-search-integration)
    1. [用户在搜索栏中输入字符](#1-user-enters-character-in-search-bar)
    2. [调用DNA SDK获取所有搜索结果](#2-call-the-dna-sdk-to-get-all-search-results)
    3. [为2种场景拆分DNA结果](#3-split-out-the-dna-results-for-the-2-scenarios)
    4. [显示DNA的所有场景5结果](#4-show-all-scenario-5-results-from-dna)
    5. [为所有可见的场景5结果注册展示](#5-register-impressions-for-all-visible-scenario-5-results)
    6. [将场景6结果与vivo有机结果混合](#6-mix-scenario-6-results-with-vivo-organic-results)
    7. [为所有可见的场景6结果注册展示](#7-fire-impressions-for-all-visible-scenario-6-results)
    8. [在UI中加载广告创意](#8-load-ad-creative-in-the-ui-for-either-scenario)
    9. [为任一场景将用户点击发送给DNA进行路由](#9-send-user-click-to-dna-for-routing-for-either-scenario)

# DNA SDK Integration
DNA SDK 集成

See [vivo-sdk-changelog](vivo-sdk-changelog.md) for release details.

### 1. Add AAR Dependency
1 添加 AAR 依赖

The DeviceNativeAds SDK is distributed as an AAR file. Follow the instructions below to install it.
DeviceNativeAds SDK 以 AAR 文件的形式分发。请按照以下说明进行安装。

#### 1.1. Download the AAR File
1.1. 下载 AAR 文件

You can find the latest AAR hosted here: [https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.6.aar](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.6.aar)

#### 1.2 Place the AAR File in your Project
1.2 将 AAR 文件放置在您的项目中

Place the DeviceNativeAds SDK in the `libs` folder of your Android project. If you don't have a `libs` folder, create one. It should be placed in the same folder as your `src` folder like so:

将 DeviceNativeAds SDK 放在 Android 项目的 `libs` 文件夹中。如果您没有 `libs` 文件夹，请创建一个。它应该与您的 `src` 文件夹放在同一目录下，如下所示：

```
project-folder/src/main/java/com/example/project/MainActivity.java
project-folder/libs/com.devicenative.dna-vivo-v1.1.6.aar
```

#### 1.3 Add the AAR Dependency
1.3 添加 AAR 依赖

Add the following dependency to your app's `build.gradle` file:

将以下依赖项添加到您应用的 `build.gradle` 文件中：

```gradle
dependencies {
    implementation files('libs/com.devicenative.dna-vivo-v1.1.6.aar')
}
```

or some Gradle versions:

或者对于某些 Gradle 版本：

```gradle
dependencies {
    implementation(files('libs/com.devicenative.dna-vivo-v1.1.6.aar'))
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
    dna.init("da01177b-1526-4b81-9b9d-4a24e54674ac");

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

# GS Recommended Apps Integration
GS 推荐应用集成

### Introduction
介绍

This section of the documentation will describe the technical integration for the following integration scenarios:

本文档部分将描述以下集成场景的技术集成：

1. Scenario 1: Recommended Apps Slot 3 and 8, CPC
    - 场景1：推荐应用位置3和8，CPC
2. Scenario 2: Recommended Apps Slot 4 and 9, CPC/CPA
    - 场景2：推荐应用位置4和9，CPC/CPA
3. Scenario 3: Recommended Apps Slot 5 and 10, CPA
    - 场景3：推荐应用位置5和10，CPA

It is important to understand that **DNA returns ALL suggestions (re-engagement and install) in a single SDK function call**. Vivo will make this function call, and then parse out the results to display in the different scenarios. DNA can assist in writing this logic for vivo.

重要的是要理解，**DNA在单个SDK函数调用中返回所有建议（重新参与和安装）**。vivo将进行此函数调用，然后解析结果以在不同场景中显示。DNA可以协助vivo编写此逻辑。

### 1: User opens Global Search app
1. 用户打开全局搜索应用

Adding this step to indicate that the user has opened the Global Search app, which is the trigger for the following logic.

添加此步骤以表明用户已打开全局搜索应用，这是以下逻辑的触发器。

### 2. Call the DNA SDK to get ALL recommended apps
2 调用DNA SDK获取所有推荐应用

This method call will return a list of DNAResultItem objects which will be used for Scenario 1, 2, and 3. It will return 6 total results which can be used for the 6 available slots.

此方法调用将返回一组 DNAResultItem 对象，用于场景1、2和3。它将返回可用于 6 个可用槽位的 6 个总结果。

```java
DeviceNativeAds dna = DeviceNativeAds.getInstance(getApplicationContext());
List<DNAResultItem> adUnits = dna.getAdsForCache(6, "gs, recommended apps");
```

#### Note
注意

1. These are ordered by relevance and monetization potential, so the first ad will be the most relevant.
    - 这些是根据相关性和货币化潜力排序的，所以第一个广告是最相关的。
2. This method return an ad in milliseconds, so it's safe to run on the main thread. 
    - 此方法在毫秒内返回广告，因此可以在主线程上运行。
3. This method does NOT assume that you will show the ads, and therefore will not fire impressions. You must fire impressions yourself with a separate call.
    - 此方法不假设您会显示广告，因此不会触发展示。您必须使用单独的调用触发展示。
4. The placement tag "global search recommended apps" is used to identify the request for the ads, but you will separately tag the ads later with a new placement tag when registering impressions.
    - 使用 placement tag "global search recommended apps" 来识别广告请求，但您稍后会使用新的 placement tag 注册广告时，会分别标记广告。
5. You can separate re-engagement and install ads using the boolean `isInstalled` field.
    - 您可以使用布尔值 `isInstalled` 字段来区分重新参与和安装广告。

#### Key Fields of DNAResultItem Class


| Parameter | Description | Translation |
|----------|-------------|-------------|
| `id` | Unique identifier for the ad. Just a UUID for reference if you need | 广告的唯一标识符。如果需要,只是一个用于参考的UUID |
| `packageName` | The package name of the advertiser's app | 广告主应用的包名 |
| `isInstalled` | A convenient boolean indicating whether the advertiser's app is installed, derived from package manager | 一个方便的布尔值,表示广告主的应用是否已安装,从包管理器派生 |
| `appName` | The name of the advertiser's app | 广告主应用的名称 |
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

In this step, vivo will retrieve its own list of recommended apps. This is the code that currently powers the "Recommended Apps" section of the Global Search app.

在此步骤中，vivo将检索其自己的推荐应用列表。这是当前为全局搜索应用的“推荐应用”部分提供动力的代码。

### 4. Split out the DNA results for the 3 scenarios
4 为3种场景拆分DNA结果

As mentioned above, all of the results for the 3 scenarios are in the list of DNAResultItem objects returned from step 2. You will need to parse out the results for each scenario.

如上所述，所有 3 种场景的结果都在从第 2 步返回的 DNAResultItem 对象列表中。您需要解析每个场景的结果。

```java
List<DNAResultItem> scenario1Ads = new ArrayList<>(); // Scenario 1: Recommended Apps, slot 3 and 8, CPC 
List<DNAResultItem> scenario2Ads = new ArrayList<>(); // Scenario 2: Recommended Apps, slot 4 and 9, CPC+ CPA
List<DNAResultItem> scenario3Ads = new ArrayList<>(); // Scenario 3: Recommended Apps, slot 5 and 10, CPA

// Note this is a reference to adUnits from step 2
// 注意，这是从第 2 步的 adUnits 的引用
for (DNAResultItem resultItem : adUnits) {
  if (resultItem.isInstalled) {
    boolean willInsert = false;
    // Candidate for Scenario 1 or 2
    // 候选场景1或2
    if (scenario1Ads.size() == 0) {
      willInsert = true;
      scenario1Ads.add(resultItem);
    } else if (scenario2Ads.size() == 0) {
      willInsert = true;
      scenario2Ads.add(resultItem);
    } else if (scenario1Ads.size() == 1) {
      willInsert = true;
      scenario1Ads.add(resultItem);
    } else if (scenario2Ads.size() == 1) {
      willInsert = true;
      scenario2Ads.add(resultItem);
    }

    if (willInsert) {
      // You should remove the app from the original vivo organic to prevent duplicates
      // This function would be implemented by vivo elsewhere
      // 如果存在，从 vivo 的有机结果中删除该应用
      // vivo 会在其他地方实现此功能
      removeFromOrganicVivoResultsIfPresent(resultItem.packageName);
    }
  } else {
    // Candidate for Scenario 3
    // 候选场景3
    if (scenario2Ads.size() == 0) {
      scenario2Ads.add(resultItem);
    } else if (scenario3Ads.size() == 0) {
      scenario3Ads.add(resultItem);
    } else if (scenario2Ads.size() == 1) {
      scenario2Ads.add(resultItem);
    } else if (scenario3Ads.size() == 1) {
      scenario3Ads.add(resultItem);
    }
  }
}
```

After parsing out the DNA results, you will have 3 lists of DNAResultItem objects, one for each scenario. You will then need to backfill if any of the slots are empty from vivo's organic results.

解析 DNA 结果后，您将获得 3 个 DNAResultItem 对象列表，每个场景一个。然后，如果任何槽为空，您将需要从 vivo 的有机结果中进行回填。

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

if (resultItem.description == null || resultItem.description.isEmpty()) {
  itemDescription.setVisibility(View.GONE);
} else {
  itemDescription.setText(resultItem.description);
}
```

### 6. Fire impressions for the ads with the new placement tag
6 使用新的位置标签为广告触发展示

It is important that you fire impressions for the ads when they are shown to the user. This is how DNA tracks the performance of the ads, but also manages frequency caps, targeting and many other functions.

当向用户显示广告时，重要的是触发展示。这是 DNA 跟踪广告性能的方式，但也管理频率上限、定位和许多其他功能。

```java
DeviceNativeAds dna = DeviceNativeAds.getInstance(getApplicationContext());
dna.fireImpressions(scenario1Ads, "gs, scenario 1, slot 3 and 8, CPC");
dna.fireImpressions(scenario2Ads, "gs, scenario 2, slot 4 and 9, CPC+CPA");
dna.fireImpressions(scenario3Ads, "gs, scenario 3, slot 5 and 10, CPA");
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

1. We recommend that you add some sort of "loading" UI after the user clicks a result. In some cases, there are delays in the click tracking and routing due to network latency.
    - 我们建议在用户点击结果后添加某种“加载”UI。在某些情况下，由于网络延迟，点击跟踪和路由会出现延迟。
2. Make sure to handle the failure cases, such as no activity found to handle the click, and errors in the click tracking and routing. This should be very rare, but it's important to handle it gracefully.
    - 确保处理故障情况，例如没有活动可以处理点击，以及点击跟踪和路由中的错误。这种情况应该很少见，但重要的是要优雅地处理它。

### 8. Do not cache the results
8 不要缓存结果

Fresh ad retrieval is very low latency and not resource intensive, so there is no need to cache the results. Results can be become stale very quickly, and it's important that the user sees the most relevant results for optimum CPM.

新鲜的广告检索延迟非常低，不占用资源，因此无需缓存结果。结果可能会很快变得过时，用户看到最相关的结果对于最佳 CPM 至关重要。

# GS Search Integration
GS 搜索集成

### Introduction
介绍

This section of the documentation will describe the technical integration for the following integration scenarios:

本文档部分将描述以下集成场景的技术集成：

1. Scenario 5: Search/Local Apps Result, CPC (default 3 results)
    - 场景5：搜索/本地应用结果，CPC（默认3个结果）
2. Scenario 6: Search/App Store Result: CPA (max 4 results)
    - 场景6：搜索/应用商店结果：CPA（最多4个结果）

It is important to understand that **DNA returns ALL search results (re-engagement and install) in a single SDK function call**. Vivo will make this function call, and then parse out the results to display in the different scenarios. DNA can assist in writing this logic for vivo.

重要的是要理解，**DNA在单个SDK函数调用中返回所有搜索结果（重新参与和安装）**。vivo将进行此函数调用，然后解析结果以在不同场景中显示。DNA可以协助vivo编写此逻辑。

### 1. User enters character in search bar
1 用户在搜索栏中输入字符

Adding this step to indicate that the user has entered a character in the search bar, which is the trigger for the following logic.

添加此步骤以表明用户已在搜索栏中输入字符，这是以下逻辑的触发器。

### 2. Call the DNA SDK to get ALL search results
2 调用DNA SDK获取所有搜索结果

This method call will return a list of DNAResultItem objects which will be used for Scenario 5 and 6.

此方法调用将返回一组 DNAResultItem 对象，用于场景5和6。

```java
DeviceNativeAds dna = DeviceNativeAds.getInstance(getApplicationContext());
List<DNAResultItem> adUnits = dna.getOrganicResultsForSearchForCache(query, "gs, search ads");
```

### 3. Split out the DNA results for the 2 scenarios
3 为2种场景拆分DNA结果

As mentioned above, all of the results for the 2 scenarios are in the list of DNAResultItem objects returned from step 2. You will need to parse out the results for each scenario.

如上所述，所有 2 种场景的结果都在从第 2 步返回的 DNAResultItem 对象列表中。您需要解析每个场景的结果。

```java
ArrayList<DNAResultItem> scenario5Results = new ArrayList<>(); // Scenario 5: Search/Local Apps Result, CPC (default 3 results)
ArrayList<DNAResultItem> scenario6Ads = new ArrayList<>(); // Scenario 6: Search/App Store Result: CPA (max 4 results)

for (DNAResultItem resultItem : adUnits) {
  if (resultItem.isInstalled) {
    // Candidate for Scenario 5 (local app results)
    // 候选场景5（本地应用结果）
    scenario5Results.add(resultItem);
  } else {
    // Candidate for Scenario 6 (uninstalled apps)
    // 候选场景6（未安装的应用）
    scenario6Results.add(resultItem);
  }
}
```

### 4. Show all scenario 5 results from DNA
4 显示DNA的所有场景5结果

The local app results from DNA are both Organic and Ads, ranked by relevance and monetization potential. You will replace all the original vivo local app results with the DNA results for Scenario 5. It is fine to just show 3 by default, and the rest when the user expands the search results.

DNA 的本地应用结果既是有机的，也是广告的，按相关性和货币化潜力排序。您将使用场景5的DNA结果替换所有原始的 vivo 本地应用结果。默认情况下只显示 3 个是可以的，当用户展开搜索结果时显示其余结果。

### 5. Register impressions for all visible Scenario 5 results
5 为所有可见的场景5结果注册展示


It is important that you fire impressions for the ads when they are shown to the user. This is how DNA tracks the performance of the ads, but also manages frequency caps, targeting and many other functions. The example below shows how to fire impressions for the first 3 results in scenario5Results.

当向用户显示广告时，重要的是触发展示。这是 DNA 跟踪广告性能的方式，但也管理频率上限、定位和许多其他功能。下面的示例显示了如何为 scenario5Results 中的前 3 个结果触发展示。

```java
DeviceNativeAds dna = DeviceNativeAds.getInstance(getApplicationContext());
// Filter the first 3 results in scenario5Results and fire impressions
// 过滤 scenario5Results 中的前 3 个结果并触发展示
List<DNAResultItem> visibleScenario5Results = scenario5Results.stream()
    .limit(3)
    .collect(Collectors.toList());
dna.fireImpressions(visibleScenario5Results, "gs, scenario 5, local apps");
```

### 6. Mix scenario 6 results with vivo organic results
6 将场景6的结果与 vivo 有机结果混合

DNA install ads for search are **NOT** a full replacement for vivo VStore search results. They are only a limited set of available ads. If you want to provide a high quality product, we recommend that you mix the DNA results with the vivo organic results for Scenario 6.

DNA 搜索的安装广告**不是** vivo VStore 搜索结果的完全替代品。它们只是一组有限的可用广告。如果您想提供高质量的产品，我们建议您将 DNA 结果与 vivo 有机结果混合以用于场景6。

Here is an example of how to mix the DNA results with the vivo organic results:

以下是如何将 DNA 结果与 vivo 有机结果混合的示例：

```java
List<Object> finalResults = new ArrayList<>();

// Add up to 4 DNA results
// 添加最多4个DNA结果
int dnaResultsToAdd = Math.min(scenario6Ads.size(), 4);
finalResults.addAll(scenario6Ads.subList(0, dnaResultsToAdd));

// Add vivo organic results, ensuring we don't add duplicates
// 添加 vivo 有机结果，确保不添加重复项
for (Object vivoResult : vivoOrganicResults) {
  if (finalResults.size() >= 4) break; // Stop if we already have 4 results 
  // 如果已经达到4个结果，则停止
  
  // Check if this vivo result is already in finalResults (avoid duplicates)
  // 检查这个 vivo 结果是否已经在 finalResults 中（避免重复）
  boolean isDuplicate = finalResults.stream()
    .anyMatch(item -> getPackageName(item).equals(getPackageName(vivoResult)));
  
  if (!isDuplicate) {
    finalResults.add(vivoResult);
  }
}

// If we still have fewer than 4 results, we can add more from vivo's organic results
// 如果我们仍然少于4个结果，我们可以从 vivo 的有机结果中添加更多
while (finalResults.size() < 4 && vivoOrganicResults.size() > finalResults.size()) {
  finalResults.add(vivoOrganicResults.get(finalResults.size()));
}
```

And the helper function to get the package name from the result object:

以及从结果对象获取包名的辅助函数：

```java
private String getPackageName(Object result) {
    if (result instanceof DNAResultItem) {
        return ((DNAResultItem) result).packageName;
    } else if (result instanceof VivoResultItem) { // Assume VivoResultItem is vivo's result class
        return ((VivoResultItem) result).getPackageName();
    }
    // Add more conditions for other result types if needed
    // 如果需要，请添加更多条件以处理其他结果类型
    return "";
}
```

### 7. Fire impressions for all visible Scenario 6 results
7 为所有可见的场景6结果注册展示

It is important that you fire impressions for the ads when they are shown to the user. This is how DNA tracks the performance of the ads, but also manages frequency caps, targeting and many other functions. The example below shows how to fire impressions for the first 4 results in scenario6Ads.

当向用户显示广告时，重要的是触发展示。这是 DNA 跟踪广告性能的方式，但也管理频率上限、定位和许多其他功能。下面的示例显示了如何为 scenario6Ads 中的前 4 个结果触发展示。

```java
DeviceNativeAds dna = DeviceNativeAds.getInstance(getApplicationContext());
// Filter the first 4 results in scenario6Ads and fire impressions
// 过滤 scenario6Ads 中的前 4 个结果并触发展示
List<DNAResultItem> visibleScenario6Results = scenario6Ads.stream()
    .limit(4)
    .collect(Collectors.toList());
dna.fireImpressions(visibleScenario6Results, "gs, scenario 6, app store");
```

### 8. Load ad creative in the UI for either scenario
8 在UI中加载广告创意

Below shows an example implementation of loading the ad creative in the UI, with DNA method calls. (This is the same as the Recommended Apps section in the Global Search app.)

以下显示了在 UI 中加载广告创意的示例实现，使用 DNA 方法调用。（这与 Global Search 应用中的推荐应用部分相同。）

```java
ImageView itemIcon = itemView.findViewById(R.id.item_icon);
TextView itemTitle = itemView.findViewById(R.id.item_title);
TextView itemDescription = itemView.findViewById(R.id.item_description);

// handle loading app icon async if app uninstalled
// 异步处理加载应用图标（如果应用未安装）
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

if (resultItem.description == null || resultItem.description.isEmpty()) {
  itemDescription.setVisibility(View.GONE);
} else {
  itemDescription.setText(resultItem.description);
}
```

#### 8.1 Access app ratings, downloads, and reviews
8.1 访问应用评分、下载次数和评论

You can access the ratings, downloads, and reviews of the advertiser's app from Google Play by using the `ratings`, `downloads`, and `reviews` fields in the `DNAResultItem` object.

您可以通过使用 `DNAResultItem` 对象中的 `ratings`、`downloads` 和 `reviews` 字段来访问广告主应用在 Google Play 上的评分、下载次数和评论。

### 9. Send user click to DNA for routing for either scenario
9 将用户点击发送给DNA进行路由

After the user clicks on a DNA result, vivo will send the click to DNA for routing. DNA should handle the click routing because it is important to deep link the user to the advertiser's app with the appropriate parameters. (This is the same as the Recommended Apps section in the Global Search app.)

用户点击 DNA 结果后，vivo 将点击发送给 DNA 进行路由。DNA 应该处理点击路由，因为重要的是使用适当的参数将用户深度链接到广告主的应用。（这与 Global Search 应用中的推荐应用部分相同。）

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

1. We recommend that you add some sort of "loading" UI after the user clicks a result. In some cases, there are delays in the click tracking and routing due to network latency.
    - 我们建议您在用户点击结果后添加某种“加载”UI。在某些情况下，由于网络延迟，点击跟踪和路由会出现延迟。
2. Make sure to handle the failure cases, such as no activity found to handle the click, and errors in the click tracking and routing. This should be very rare, but it's important to handle it gracefully.
    - 确保处理故障情况，例如没有活动可以处理点击，以及点击跟踪和路由中的错误。这种情况应该很少见，但重要的是要优雅地处理它。