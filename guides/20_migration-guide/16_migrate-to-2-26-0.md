---
description: Documentation for Migrate To 2 26 0
title: Migrate To 2 26 0
---

# Migrate to 2.26.0

This guide outlines the breaking changes introduced in SDK version 2.26.0. Required updates may vary depending on your use case.

Additionally, new features and bugfixes have been introduced and are **not** documented here. For a comprehensive list of changes, please refer to the changelog.

This release adds new marker and navigation features, removes unstable classes, refines method signatures and return types, and fixes crashes with destroyed controllers.

## The *calculateRoute* method from the *calculateRoute* class now takes the dimensions from *RoutePreferences.truckProfile* object even if the transport mode is *RouteTransportMode.car*

Previously, if the `transportMode` was set to `RouteTransportMode.car`, the dimensions and weights from `RoutePreferences.truckProfile` were not taken into account.
Now, if the `transportMode` is set to `RouteTransportMode.car`, the dimensions from `RoutePreferences.truckProfile` will be taken into account.

This allows options for caravan routes, allowing you to specify vehicle dimensions without being limited to truck routes. The `transportMode` field is essential for distinguishing a truck from other types of vehicle.

## Removed the *Activity* class and related enums and methods

The following enums and the `Activity` class were removed because they were unstable and were not fully supported:

- `Activity` class

- `ActivityType`, `ActivityConfidence` enums

- `activity` value from the `DataType` enum

- `produceActivity` method from the `SenseDataFactory` class

If your code referenced `Activity` or any of the removed enums, remove those references and migrate to the alternative telemetry or sensor outputs that your project uses. There is no direct one-to-one replacement for `Activity`.

## Made properties internal in the *MarkerRenderSettings* class

Affected members are `imagePointer`, `packedLabelingMode`, `imagePointerSize`

These properties were made internal in `MarkerRenderSettings` API as they were not supposed to be exposed as part of the public API.
If you relied on them, you must now use the other `MarkerRenderSettings` members.

## Made *create* method internal in the *TimezoneResult* class

`TimezoneResult` instances are designed to be obtained via `TimezoneService` operations rather than via `create` methods.

## Removed *scroll* methods from *GemMapController* and *GemView*

The affected classes are `GemMapController` and `GemView`.

This low-level scroll helpers were removed because they were non-functional.

## The *getPlayback* method was replaced by *playback* getter in *PositionService*

The `getPlayback` method has been replaced by a `playback` getter for simpler, idiomatic access.

Before:
```dart
Playback? playback = await positionService.getPlayback();
```

After:
```dart
Playback? playback = positionService.playback;
```

## The *getCountryData* method from *MapDetails* returns nullable *CountryData*

The method previously returned `CountryData` and now returns `CountryData?`. Callers must handle missing country data appropriately.

Before:
```dart
CountryData country = MapDetails.getCountryData(12);
if (ApiErrorService.apiError != GemError.success){
    // do something with the country data
} else {
    // // handle no country available for the given index
}
```

After:
```dart
CountryData? country = mapDetails.getCountryData(code);
if (country != null) {
    // do something with the country data
} else {
    // handle no country available for the given index
}
```

## Removed redundant *type* parameter from the *setMockData* method from the *DataSource* class

The `type` parameter was redundant and removed. Callers should remove that argument as it can be obtained internally based on the passed data.

Before:
```dart
GemPosition position = ...
dataSource.setMockData(position, DataType.position);
```

After:
```dart
GemPosition position = ...
dataSource.setMockData(position);
```

## The *area* parameter of the *hitTest* method from *MapViewMarkerCollections* was replaced with a *coordinates* parameter

The `hitTest` method no longer accepts an `area` argument; instead it accepts a `coordinates` parameter that improves precision by using actual coordinates for the hit test.

Before this change, the `RectangleGeographicArea` passed as the argument needed to have a `topLeft` identical to the `bottomRight` coordinate for the hit test to work correctly, as the method was effectively performing a point hit test.
The new `coordinates` parameter makes this explicit and avoids confusion.

Before:
```dart
MapViewMarkerCollections mapViewMarkerCollections = ...

Coordinates coords = Coordinates(latitude: 53.57, longitude: -119.77);
RectangleGeographicArea area = RectangleGeographicArea(topLeft: coords, bottomRight: coords);

final hits = mapViewMarkerCollections.hitTest(area);
```

After:
```dart
MapViewMarkerCollections mapViewMarkerCollections = ...

Coordinates coords = Coordinates(latitude: 53.57, longitude: -119.77);

final hits = mapViewMarkerCollections.hitTest(coords);
```

## The method *getBestLanguageMatch* from *SdkSettings* has changed and has been fixed

The method now returns `Language?` instead of `Language`. The `variant` parameter type changed from `int` to `ScriptVariant`.
Additionally, `regionCode`, `scriptCode`, and `variant` are now named parameters with default values.

The method was also fixed, previously it did not function correctly.

Before:
```dart
Language lang = sdkSettings.getBestLanguageMatch('eng', 'GBR', '', 0);
if (lang.name.isNotEmpty) {
	// Use the language
} else {
    // No language found
}
```

After:
```dart
final Language? lang = sdkSettings.getBestLanguageMatch(
	languageCode: 'en',
	regionCode: 'GBR',
	variant: ScriptVariant.native,
);
if (lang != null) {
	// Use the language
} else {
    // No language found
}
```

If you previously passed a positional `variant` `int`, convert it to the `ScriptVariant` enum value. If you omitted the new named parameters, the SDK will use the provided defaults.

## Removed the *TrafficTransportMode* enum

This enum was accidentally left in the public API but was never fully supported or documented. It has been removed to avoid confusion.

## Other enum updates

- Added `free` to `TrafficEventSeverity`

- Added `waitingReturnToRoute` to `NavigationStatus`

- Added `textureView` to `AndroidViewMode`

- Added `groupTopRight` to `MarkerLabelingMode`

- Added `packedGeometry` and `polyline` to `MarkerLabelingMode`

- Removed `activity` from `DataType`

These are additive/removal enum changes. Add cases where you `switch` over these enums to handle the new values and remove handling for `DataType.activity`.
