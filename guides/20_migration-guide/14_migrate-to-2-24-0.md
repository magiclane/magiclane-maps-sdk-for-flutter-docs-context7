---
description: Documentation for Migrate To 2 24 0
title: Migrate To 2 24 0
---

# Migrate to 2.24.0

This guide outlines the breaking changes introduced in SDK version 2.24.0. Required updates may vary depending on your use case.

Additionally, new features and bugfixes have been introduced and are **not** documented here. For a comprehensive list of changes, please refer to the changelog.

This release introduces new camera support and rendering behavior updates, deprecates some legacy fields and methods, and improves consistency in return types across several APIs.

## The  *onCompleteCallback* parameter replaced with *onComplete*

The `onCompleteCallback` parameter has been deprecated across all classes and replaced with `onComplete`.

Before:
```dart
SdkClass.method(onCompleteCallback: (...) {
  // Callback logic
});
```

After:
```dart
SdkClass.method(onComplete: (...) {
  // Callback logic
});
```

Affected classes and methods are:

| Class                      | Methods                                                                             | Observation                                                                                                             |
|----------------------------|-------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------|
| WeatherService             | getCurrent, getForecast, getHourlyForecast, getDailyForecast                        | Provides the GemError and result                                                                                        |
| RouteTrafficEvent          | asyncUpdateToFromData                                                               | Provides GemError                                                                                                       |
| TimezoneService            | getTimezoneInfoFromCoordinates, getTimezoneInfoFromTimezoneId                       | Provides GemError and TimezoneResult?                                                                                   |
| SearchService              | search, searchLandmarkDetails, searchAlongRoute, searchInArea, searchAroundPosition | Provides GemError and list of Landmark                                                                                  |
| RoutingService             | calculateRoute                                                                      | Provides GemError and list of Route                                                                                     |
| ProjectionService          | convert                                                                             | Provides GemError and Projection?                                                                                       |
| LandmarkStore              | importLandmarks, importLandmarksWithDataBuffer                                      | Provides GemError                                                                                                       |
| GuidedAddressSearchService | search, searchCountries                                                             | Provides GemError and list of Landmark                                                                                  |
| ContentUpdater             | update                                                                              | Provides GemError                                                                                                       |
| ContentStoreItem           | asyncDownload                                                                       | Provides GemError                                                                                                       |
| ContentStore               | asyncGetStoreFilteredList, asyncGetStoreContentList                                 | Provides GemError and list of ContentStoreItem. In case of asyncGetStoreContentList also returns a isCaches boolean value |

This change aligns with common Dart naming conventions and enhances clarity.

## Angle names are now uniform in the SDK

Different names were used for the same thing through the SDK. The current release standardizes the naming.

### The *headingInDegrees* replaced with *mapAngle* in *MapView*

The `headingInDegrees` getter has been deprecated. Use `mapAngle` from `MapView` instead.

Before:
```dart
MapView mapView = ...
double heading = mapView.headingInDegrees;
```

After:
```dart
MapView mapView = ...
double heading = mapView.mapAngle;
```

### The *pitchInDegrees* replaced with *viewAngle* in *MapView*

The `pitchInDegrees` member has been deprecated. Use `viewAngle` instead.

Before:
```dart
MapView mapView = ...
double pitch = mapView.pitchInDegrees;
```

After:
```dart
MapView mapView = ...
double pitch = mapView.viewAngle;
```

### The *angle* parameter of *setMapRotationMode* replaced with *mapAngle* in *FollowPositionPreferences*

The `angle` parameter has been deprecated in favor of `mapAngle`.

Before:
```dart
FollowPositionPreferences prefs = ...
prefs.setMapRotationMode(..., angle: 45.0);
```

After:
```dart
FollowPositionPreferences prefs = ...
prefs.setMapRotationMode(..., mapAngle: 45.0);
```

### The *rotationAngle* replaced with *mapAngle* in *FollowPositionPreferences* and *MapViewPreferences*

The `rotationAngle` member has been deprecated. Use `mapAngle` instead.

Before:
```dart
MapViewPreferences prefs = ...
double angle = prefs.rotationAngle;
prefs.rotationAngle = 90.0;
```

After:
```dart
MapViewPreferences prefs = ...
double angle = prefs.mapAngle;
prefs.mapAngle = 90.0;
```

## The *isEmpty* getter is replaced with *isDefault*  in all *GeographicArea* classes

To better reflect intent, the `isEmpty` getter is now replaced with `isDefault`.

Before:
```dart
bool empty = area.isEmpty;
```

After:
```dart
bool isDefault = area.isDefault;
```

## Return types in `RouteBookmarks` changed to nullable

To improve robustness and null safety, return types of multiple methods in the `RouteBookmarks` class have been changed from non-nullable to nullable.

| Method           | Old Return Type     | New Return Type    |
|------------------|---------------------|---------------------|
| `getWaypoints`   | `List<Landmark>`    | `List<Landmark>?`   |
| `getName`        | `String`            | `String?`           |
| `getTimestamp`   | `DateTime`          | `DateTime?`         |

The value null is now returned when the provided index is invalid.

Before:
```dart
final name = bookmarks.getName(-1);
if (name.isNotEmpty){
  // Do something with name
} else {
  // The index is invalid
}
```

After:
```dart
final name = bookmarks.getName();
if (name != null) {
  /// Do something with name
} else {
  /// The index is invalid
}
```

This change improves clarity by explicitly indicating that the returned result is invalid.

## Simplified rendering API

Manual rendering is no longer available. The `render` and `markNeedsRender` methods from the `GemView` and `GemMapController` classes have been removed.
If you were manually invoking these methods, you can now safely remove these calls.

The `renderingRule` property of the `GemView` and `GemMapController` classes has been replaced with the `isRenderEnabled` getter/setter pair. The `RenderRule` enum has also been removed.
The newly added member removes the platform specific logic.

Before:
```dart
GemMapController controller = ...
controller.renderingRule = RenderRule.noRender; // Stop render on iOS/Android.
controller.renderingRule = RenderRule.automatic; // Start render on iOS only.
controller.renderingRule = RenderRule.onDemand; // Start render on Android.
```

After:
```dart
GemMapController controller = ...
controller.isRenderEnabled = false; // Stop render on iOS/Android.
controller.isRenderEnabled = true; // Start render on iOS/Android.
```

Visual glitches may still persist on older Android versions when enabling rendering.

## The `accurateResult` parameter has been removed from timezone methods

The optional `accurateResult` parameter has been removed from:

- `Timezone.getTimezoneInfoFromCoordinates`

- `Timezone.getTimezoneInfoFromTimezoneId`

Before:
```dart
timezone.getTimezoneInfoFromCoordinates(coords, accurateResult: true);
```

After:
```dart
timezone.getTimezoneInfoFromCoordinates(coords);
```

The methods now automatically return the best available result, equivalent to when `accurateResult` is set to true.

To retrieve the result synchronously without making a server request (i.e., the behavior when `accurateResult` is false), use the newly added `getTimezoneInfoFromCoordinatesSync` and `getTimezoneInfoFromTimezoneIdSync` methods.
