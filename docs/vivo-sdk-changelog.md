---
hide:
  - navigation
  - toc
---

# vivo DNA SDK Changelog

## **vivo-1.1.0** released 8/19/2024

- First release of custom vivo DNA SDK
- Performance optimizations for vivo Global Search use case

## **vivo-1.1.1** released 8/20/2024

- Added ability to register impression manually for DNA search results (already existed for recommendations)

## **vivo-1.1.2** released 8/22/2024

- Remote control to disable app usage stats collection
- Improved poor connection handling in link routing use case
- Exposed app ratings, downloads, and reviews in DNAResultItem

## **vivo-1.1.3** released 8/23/2024

- Redirect prefetching for high likelihood clicks
- Cache and reused redirect outcomes for improved performance on repeat clicks
- Improved thread performance for click speedup
- Implemented deep linking for install ads that are already installed

## **vivo-1.1.4** released 8/26/2024

- Most app-install clicks will now be instantaneous
- Parallel process tracking link and Play Store redirect when possible
- Improved performance of Google Play to Custom Store remapping

## **vivo-1.1.5** released 9/5/2024

- Initialization ongoing memory reduction of ~ 20MB
- Isolated Chrome User Agent fetch to process in order to constrain memory consumption

## **vivo-1.1.6** released 9/9/2024

- Made DNAResultItem Parceable

## **vivo-1.1.7** released 9/26/2024

- Improved the robustness of inter-process communication for user agent
- Lower resource consumption when reading user agent
- Added code to prevent duplicate data orchestrator services from running