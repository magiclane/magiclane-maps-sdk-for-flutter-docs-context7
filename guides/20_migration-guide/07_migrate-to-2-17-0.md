---
description: Documentation for Migrate To 2 17 0
title: Migrate To 2 17 0
---

# Migrate to 2.17.0

This guide outlines the breaking changes introduced in SDK version 2.17.0. Required updates may vary depending on your use case.

Additionally, new features and bugfixes have been introduced and are **not** documented here. For a comprehensive list of changes, please refer to the changelog.

This release brings an overhaul related to images and adds more methods used for debugging.

## Image changes

This release adds the `LaneImg`, `AbstractGeometryImg`, `SignpostImg`, `RoadInfoImg`, `Img` and `ImgBase` classes which provide functionalities such as:

- Retrieving recommended image sizes and aspect ratios

- Accessing image UIDs

- Enabling the SDK to resize certain image types (`LaneImg`, `SignpostImg`, `RoadInfoImg`) based on their content

Additionally, the new `RenderableImg` class wraps raw `Uint8List` image data and provides metadata such as width and height.

See the [working with images](../core/images) page for more details about this change.

Before:
```dart
Uint8List? imageData = instruction.getLaneImage(
    size: const Size(200, 100),
    format: ImageFileFormat.png,
);
```

Now:
```dart
LaneImg laneImage = instruction.laneImg;
RenderableImg? renderableImage = newImage.getRenderableImage(size: const Size(200, 100), format: ImageFileFormat.png, allowResize : true);
Uint8List? imageData = renderableImage?.bytes;

// Get more data about the lane image
int uid = laneImage.uid;

// Get the actual size of the image
int? width = renderableImage?.width;
int? height = renderableImage?.height;
```

This change affects how images are handled throughout the SDK.

While the new image architecture is recommended, the previous method of directly retrieving `Uint8List` data remains available for backward compatibility. However, it is advised to transition to the new structure to leverage the added flexibility and metadata access.

As part of the structural changes to the `Conditions` and `OverlayCategory` classes to align with the new image architecture, the `image` setter has been removed from both classes.

## The *searchReportsAlongRoute* and *searchReportsAround* methods from the *SocialOverlay* class have been removed

Overlay items can now be searched using the methods provided by the `SearchService`. Additionally, if a landmark from the search results corresponds to an `OverlayItem`, it can be converted to an `OverlayItem` using the `overlayItem` getter.

Before:
```dart
SocialOverlay.searchReportsAround(
    position: Coordinates(latitude: 48.896680, longitude: 2.310136), // Paris
    onCompleteCallback: (GemError err, List<OverlayItemPosition> result) {
        for (final itemPos in result){
            final OverlayItem item  = itemPos.overlayItem;
            // Do something with the overlay item
        }
    },
);
```

After:
```dart
// Get the overlay id of safety
int overlayId = CommonOverlayId.safety.id;

// Add the overlay to the search preferences
SearchPreferences preferences = SearchPreferences();
preferences.overlays.add(overlayId);

// We can set searchMapPOIs and searchAddresses to false if no results from the map POIs and addresses should be returned
preferences.searchMapPOIs = false;
preferences.searchAddresses = false;

SearchService.search(
    'Speed',
    Coordinates(latitude: 48.84202113309092, longitude: 2.3254994453015634),
    preferences: preferences,
    (GemError err, List<Landmark> result) {
        for (final lmk in result){
            final OverlayItem? item  = lmk.overlayItem;
            if (item != null){
                // Do something with the overlay item
            }
        }
    }
);
```

Make sure the `SearchPreferences` object passed to the search operation is configured to search on the correct overlays. See the [Search on overlays section](../search/get-started-search#search-on-overlays) for more details.

## The *networkProvider* getter of the *SdkSettings* class has been removed

The `networkProvider` getter in `SdkSettings` has been removed because it was non-functional.

## The *clear* method from the *MapViewMarkerCollections* class is now async

This method is now async and needs to be awaited.

Before:
```dart
MapViewMarkerCollections collection = ...
collection.clear();
```

After:
```dart
MapViewMarkerCollections collection = ...
await collection.clear();
```

## The *pauseDownload* method from the *ContentStoreItem* class now takes an optional *onComplete* parameter

It is recommended to wait the `onComplete` callback to be triggered before doing other operations on the `ContentStoreItem` object, otherwise the operation might not be taken into account.

Before:
```dart
ContentStoreItem item = ...
GemError error = item.pause();
```

After:
```dart
ContentStoreItem item = ...
GemError error = item.pause((GemError error){
    if (error == GemError.success){
        // The pause succeeded. Other operations such as asyncDownload can now be called
    } else {
        // The pause failed.
    }
});
```


