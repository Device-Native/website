---
hide:
  - navigation
  - toc
---

# vivo DNA SDK Changelog

## [**vivo-1.1.0**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.0.aar) released 8/19/2024

- First release of custom vivo DNA SDK
- Performance optimizations for vivo Global Search use case

## [**vivo-1.1.1**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.1.aar) released 8/20/2024

- Added ability to register impression manually for DNA search results (already existed for recommendations)

## [**vivo-1.1.2**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.2.aar) released 8/22/2024

- Remote control to disable app usage stats collection
- Improved poor connection handling in link routing use case
- Exposed app ratings, downloads, and reviews in DNAResultItem

## [**vivo-1.1.3**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.3.aar) released 8/23/2024

- Redirect prefetching for high likelihood clicks
- Cache and reused redirect outcomes for improved performance on repeat clicks
- Improved thread performance for click speedup
- Implemented deep linking for install ads that are already installed

## [**vivo-1.1.4**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.4.aar) released 8/26/2024

- Most app-install clicks will now be instantaneous
- Parallel process tracking link and Play Store redirect when possible
- Improved performance of Google Play to Custom Store remapping

## [**vivo-1.1.5**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.5.aar) released 9/5/2024

- Initialization ongoing memory reduction of ~ 20MB
- Isolated Chrome User Agent fetch to process in order to constrain memory consumption

## [**vivo-1.1.6**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.6.aar) released 9/9/2024

- Made DNAResultItem Parceable

## [**vivo-1.1.7**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.7.aar) released 9/26/2024

- Improved the robustness of inter-process communication for user agent
- Lower resource consumption when reading user agent
- Added code to prevent duplicate data orchestrator services from running

## [**vivo-1.1.8**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.8.aar) released 9/30/2024

- Fixed potential null pointer exception in profile change monitor

## [**vivo-1.1.9**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.9.aar) released 10/8/2024

- Added DNAResultItem.className and DNAResultItem.getComponentName to support alternative filtering/deduping
- SDK now supports work profile management
- Some performance improvements to ad serving

## [**vivo-1.1.10**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.10.aar) released 10/20/2024

- Fixed a bug where the app name was being used instead of the title
- Implemented better launching of User Agent collection service to handle background restrictions

## [**vivo-1.1.11**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.11.aar) released 11/1/2024

- Reduced the time taken to retrieve the Chrome User Agent in the separate process to minimize the chance of collisions.

## [**vivo-1.1.12**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.12.aar) released 11/5/2024

- Better management of the local GAID
- Change of direct linking for app install campaigns

## [**vivo-1.1.13**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.13.aar) released 11/7/2024

- Fixed an issue where new apps were not being assigned a user profile
- Improved the latency of link redirects in some cases
- Added support for intent-based deep linking

## [**vivo-1.1.14**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.14.aar) released 11/24/2024

- Changed back to market:// URLs for app install campaigns
- Fixed a bug for routing duplicate clicks
- Removed logging when debug mode is off
- Fast open app store when link is cached
- Query parameters for market URL rewrites

## [**vivo-1.1.15**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.15.aar) released 12/4/2024

- Added support for gzip for reduced bandwidth usage

## [**vivo-1.1.16**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.16.aar) released 12/10/2024

- Added support for destination URL override

## [**vivo-1.1.17**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.17.aar) released 12/12/2024

- More robust coverage of initialization to handle unpredicatible calls to destroy()