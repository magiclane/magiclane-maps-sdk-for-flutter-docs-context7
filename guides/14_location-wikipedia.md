---
description: Documentation for Location Wikipedia
title: Location Wikipedia
---

# Location Wikipedia

Landmarks can include Wikipedia data such as title, image title, URL, description, page summary, and language. The `ExternalInfo` class handles Wikipedia data and is provided by the `ExternalInfoService` class.

---

## Check Wikipedia Data Availability

Use the `hasWikiInfo` method to check if a landmark has Wikipedia data:
```dart
final bool hasExternalInfo = ExternalInfoService.hasWikiInfo(landmark);
```

Do not modify Wikipedia-related fields in the `extraInfo` property when changing landmark data.

---

## Get Wikipedia Information

Obtain an `ExternalInfo` object using the `requestWikiInfo` method:
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

The `requestWikiInfo` returns a progress listener that can cancel the request using the `cancelWikiInfo` method.

Wikipedia data is provided in the language specified in `SDKSettings`. See [Internationalization](/guides/get-started/internationalization) for details.

**Results:**

- **Success:** Returns `GemError.success` and a non-null `ExternalInfo` object

- **Failure:** Returns null `ExternalInfo` and one of these errors:

  - `GemError.invalidInput` - Landmark does not contain Wikipedia information

  - `GemError.connection` - No internet connection available

  - `GemError.notFound` - Wikipedia information could not be retrieved

  - `GemError.general` - Unspecified error occurred

---

## Get Wikipedia Image Data

Access image details from the `ExternalInfo` class:
```dart
final int imgCount = externalInfo.imagesCount;
final String imageUrl = externalInfo.getWikiImageUrl(0);
final String imageDescription = externalInfo.getWikiImageDescription(0);
final String imageTitle = externalInfo.getWikiImageTitle(0);
```

Retrieve detailed image information using the `requestWikiImageInfo` method:
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

The `requestWikiImageInfo` method returns a progress listener that can cancel the request using the `cancelWikiImageInfoRequest` method.

For getting the image data itself, use the `requestWikiImage` method to obtain a `Img` instance. The `ExternalImageQuality` enum allows specifying the desired image quality.
```dart
final imageDataListener = externalInfo.requestWikiImage(
  imageIndex: 0,
  quality: ExternalImageQuality.highImageQuality,
  onComplete: (GemError error, Img? image) {
    if (error != GemError.success) {
      showSnackbar("Error getting wiki image: $error");
      return;
    }

    // Do something with image...
  },
);
```

It is recommended to use the `requestWikiImageInfo` method instead of using the `getWikiImageUrl` image URL directly.
The `getWikiImageUrl` method provides a URL to the original image, which may have a very large size.

---

## Relevant example demonstrating Wikipedia-related features

- [Location Wikipedia](/docs/flutter/examples/places-search/location-wikipedia)
