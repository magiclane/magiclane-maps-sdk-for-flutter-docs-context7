---
description: Documentation for Migrate To 2 20 0
title: Migrate To 2 20 0
---

# Migrate to 2.20.0

This guide outlines the breaking changes introduced in SDK version 2.20.0. Required updates may vary depending on your use case.

Additionally, new features and bugfixes have been introduced and are **not** documented here. For a comprehensive list of changes, please refer to the changelog.

This release introduces advanced geofence alarm features, new `LandmarkStore` operations, improved handling of invalid parameters, support for user-defined roadblocks, and various bug fixes.

## Full Refactor of *ExternalInfo* API

The previous `ExternalInfo` methods were prone to errors and difficult to use. To address this, the new `ExternalInfoService` has been introduced, offering clearer and more reliable operations:

- `hasWikiInfo(Landmark)` (static) : Checks if the given `Landmark` has associated Wikipedia information. Replaces `ExternalInfo.hasWikiInfo`.

- `requestWikiInfo(Landmark)` (static) : Retrieves the `ExternalInfo` object linked to the specified `Landmark`. Replaces `ExternalInfo.getExternalInfo`.

- `cancelWikiInfo(Landmark)` (static) : Cancels an ongoing `requestWikiInfo` operation. Replaces `ExternalInfo.cancelWikiInfo`.

### Check if wiki data is available

Before:
```dart
final ExternalInfo pExternalInfo = ExternalInfo();
final bool hasExternalInfo = pExternalInfo.hasWikiInfo(landmark);
```

After:
```dart
final bool hasExternalInfo = ExternalInfoService.hasWikiInfo(landmark);
```

### Get Wiki data

Before:
```dart
Completer<ExternalInfo?> completer = Completer<ExternalInfo?>();
ExternalInfo.getExternalInfo(
    landmark,
    onWikiDataAvailable: (externalInfo) => completer.complete(externalInfo),
);
```

After:
```dart
Completer<ExternalInfo?> completer = Completer<ExternalInfo?>();
ExternalInfoService.requestWikiInfo(
    landmark,
    onComplete: (GemError err, ExternalInfo? externalInfo) => completer.complete(externalInfo),
);
```

The `GemError` is also provided using the new API.

### Many methods are now getters

Several commonly used methods in `ExternalInfo` have been refactored into property getters.

- `getWikiPageTitle()` → `wikiPageTitle`

- `getWikiPageDescription()` → `wikiPageDescription`

- `getWikiImagesCount()` → `wikiImagesCount`

- `getWikiPageLanguage()` → `wikiPageLanguage`

- `getWikiPageUrl()` → `wikiPageUrl`

Before:
```dart
final String title = externalInfo!.getWikiPageTitle();
final String content = externalInfo.getWikiPageDescription();
final String imgCount = externalInfo.getWikiImagesCount();
final String language = externalInfo.getWikiPageLanguage();
final String pageUrl = externalInfo.getWikiPageUrl();
```

After:
```dart
final String title = externalInfo!.wikiPageTitle;
final String content = externalInfo.wikiPageDescription;
final String language = externalInfo.wikiPageLanguage;
final String pageUrl = externalInfo.wikiPageUrl;
```

This improves readability and adheres to idiomatic Dart style by eliminating unnecessary parentheses.

## Improved null safety for invalid input

Many methods now return `null` when provided with invalid input such as out of bounds indexes:

| Class                    | Method                 | Previous Return Type   | Current Return Type  | Behavior Change                          |
|--------------------------|------------------------|------------------------|----------------------|------------------------------------------|
| `MarkerCollection`       | `getMarkerAt`          | `Marker`              | `Marker?`            | Returns `null` for out-of-bounds index    |
| `MarkerCollection`       | `getMarkerById`        | `Marker`              | `Marker?`            | Returns `null` if ID is not found         |
| `MarkerCollection`       | `getPointsGroupHead`   | `Marker`              | `Marker?`            | Returns `null` if ID is not found         |
| `MapViewPathCollection`  | `getPathAt`            | `Path`                | `Path?`              | Returns `null` for out-of-bounds index    |
| `MapViewPathCollection`  | `getPathByName`        | `Path`                | `Path?`              | Returns `null` if name is not found       |
| `EntranceLocations`      | `getCoordinates`       | `Coordinates`         | `Coordinates?`       | Returns `null` for out-of-bounds index    |

Before:
```dart
MarkerCollection collection = ...;
Marker marker = collection.getMarkerAt(10);

// Do something with the marker...
```

The old approach did not provide a simple way to check if the returned value is valid.

After:
```dart
MarkerCollection collection = ...;
Marker? marker = collection.getMarkerAt(10);

if (marker == null){
    print("Method has returned invalid result");
    return;
}

// Do something with the marker...
```

The API reference has also been enhanced, with clearer return types and improved documentation for methods handling invalid input. This enhances the robustness and predictability of the API.

## Changes in the rendering of images

Affected methods and classes are:

- `SignpostDetails.getImage/image` 

- `RouteInstructions.getRoadInfoImage/roadInfoImg`

Although there are no changes to the API, please review and update the UI as needed to ensure correct rendering.

Previously, when large dimensions were provided, these methods would center the image content rather than scaling it properly.  
With this update, the content now scales correctly to match the specified dimensions, ensuring more accurate rendering.

Additionally, text rendering on the generated images has been improved for better clarity and visual quality.

This change may affect the layout of displayed images if custom workarounds were previously applied.

## AlarmService and AlarmListener changes regarding the areas

The `crossedBoundaries` getter of the `AlarmService` class has been removed.
It has been replaced by a redesigned `onBoundaryCrossed` callback in the `AlarmListener` interface. The callback type has changed from `void Function()?` to `void Function(List<String> enteredAreas, List<String> exitedAreas)?`

This feature was non-functional in previous releases. The latest API changes represent a complete overhaul of how monitored areas are handled.

For implementation details and usage guidance, see the [Areas Alarm Guide](../alarms/areas-alarms).

## The *Status* enum has been renamed to *MapStatus*

This change affects the following methods:

- `registerOnWorldwideRoadMapSupportStatus` and `registerOnAvailableContentUpdate` in the `OffBoardListener` class

- `setAllowConnection` in the `SdkSettings` class

To update your code, simply replace references to the `Status` enum with `MapStatus`.

Before:
```dart
SdkSettings.offBoardListener.registerOnWorldwideRoadMapSupportStatus((Status status){
    // Do something with status
});
```

After:
```dart
SdkSettings.offBoardListener.registerOnWorldwideRoadMapSupportStatus((MapStatus status){
    // Do something with status
});
```

## The type of the *labelGroupTextSize* field from the *MarkerCollectionRenderSettings* has changed from *int* to *double*

The type of the `labelGroupTextSize` field in the `MarkerCollectionRenderSettings` class has been changed from `int` to `double` to allow for more precise text size configuration.

Before:
```dart
MarkerCollectionRenderSettings collection;
collection.labelGroupTextSize = 1;

int textSize = collection.labelGroupTextSize;
```

After:
```dart
MarkerCollectionRenderSettings collection;
collection.labelGroupTextSize = 1.0;

double textSize = collection.labelGroupTextSize;
```

## The *affectedTransportModes* getter of the *TrafficEvent* class has changed type from *Set of TrafficTransportMode* to *RouteTransportMode*

The `RouteTransportMode` enum is more consistent with other SDK components and offers better compatibility with related methods.

Before:
```dart
TrafficEvent event = ...
Set<TrafficTransportMode> transportModes = event.affectedTransportModes;
```

After:
```dart
TrafficEvent event = ...
RouteTransportMode transportMode = event.affectedTransportModes;
```

## The *requestLocationPermission* getter of the *PositionService* class has been transformed into a method

The `requestLocationPermission` has been changed from a getter to a method, as it performs an action rather than simply retrieving a value—making a method a more appropriate choice.

This change is part of ongoing work to support web platforms, which is not yet publicly released. The method can be safely ignored for now.

Before:
```dart
PositionService.requestLocationPermission;
```

After:
```dart
PositionService.requestLocationPermission();
```

## Overhaul of *NetworkProvider* class

The `NetworkProvider` class has been fully redesigned, as the previous implementation did not function as expected. The new version offers new features and changes the whole structure of the class.

## The *addlist* method of the *MapViewMarkerCollections* class is now async and needs to be awaited

The `addList` method of the `MapViewMarkerCollections` class is now asynchronous and **must** be awaited.

Before:
```dart
List<int> ids = controller.preferences.markers.addList(...);
```

After:
```dart
List<int> ids = await controller.preferences.markers.addList(...);
```

This change resolves issues related to marker addition. Failing to await the method may result in unexpected behavior.
