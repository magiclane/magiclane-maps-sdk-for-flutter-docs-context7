---
description: Documentation for Location Wikipedia
title: Location Wikipedia
---

# Location Wikipedia

Landmarks can include Wikipedia data such as title, image title, URL, description, page summary, language, and more. To demonstrate how to retrieve Wikipedia information, we introduce the `ExternalInfo` class, which handles Wikipedia data.

Objects of type `ExternalInfo` are provided by the `ExternalInfoService` class.

## Check if Wikipedia data is available

Use the static `hasWikiInfo` method of the `ExternalInfoService` class to check if a landmark has wikipedia data available:
```dart
final bool hasExternalInfo = ExternalInfoService.hasWikiInfo(landmark);
```

Make sure the wikipedia related fields from the `extraInfo` property of the `Landmark` object are not tempered with if changes are made to the landmark data.

## ExternalInfo class

This class provides Wikipedia information for a landmark. An `ExternalInfo` object is obtained using the static method `requestWikiInfo` of the `ExternalInfoService` class.
```dart
final requestListener = ExternalInfoService.requestWikiInfo(
  landmark,
  onComplete: (GemError err, ExternalInfo? externalInfo) {
    if (err != GemError.success) {
      showSnackbar("Error getting wiki info: $err");
      return;
    }

    // Data about the page
    final String title = externalInfo!.wikiPageTitle;
    final String content = externalInfo.wikiPageDescription;
    final String language = externalInfo.wikiPageLanguage;
    final String pageUrl = externalInfo.wikiPageUrl;
  },
);
```

The `requestWikiInfo` returns a progress listener which can be used to cancel the request using the `cancelWikiInfo` method of the `ExternalInfoService` class.

Wikipedia data is provided in the language specified in `SDKSettings`. Learn more about setting the SDK language [here](/guides/get-started/internationalization).

The method provides a result based on the outcome of the operation:

- On success returns `GemError.success` and a non-null `ExternalInfo` object.

- On failure returns a null `ExternalInfo` object and one of the following `GemError` values:

  - `GemError.invalidInput` : The specified landmark does not contain Wikipedia-related information.

  - `GemError.connection` : No internet connection is available.

  - `GemError.notFound` : Wikipedia information could not be retrieved for the given landmark.

  - `GemError.general` : An unspecified error occurred.

## Wikipedia image data

The `ExternalInfo` class provides the following details regarding images:
```dart
final int imgCount = externalInfo.imagesCount;
final String imageUrl = externalInfo.getWikiImageUrl(0);
final String imageDescription = externalInfo.getWikiImageDescription(0);
final String imageTitle = externalInfo.getWikiImageTitle(0);
```

Detailed informations about an image can be retrieved using the `requestWikiImageInfo` method:
```dart
final imageInfoListener = externalInfo.requestWikiImageInfo(
  imageIndex: 0,
  onComplete: (GemError error, String? imageInfo) {
    if (error != GemError.success) {
      showSnackbar("Error getting wiki image info: $error");
      return;
    }

    // Do something with image info...
  },
);
```

The `requestWikiImageInfo` method return a progress listener which can be used to cancel the request using the `cancelWikiImageInfoRequest` method.

## Relevant example demonstrating Wikipedia-related features

- [Location Wikipedia](/docs/flutter/examples/places-search/location-wikipedia)
