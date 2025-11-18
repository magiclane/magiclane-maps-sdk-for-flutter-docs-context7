---
description: Documentation for Migrate To 3 1 3
title: Migrate To 3 1 3
---

# Migrate to 3.1.3

This guide outlines the breaking changes introduced in SDK version 3.1.3. Required updates may vary depending on your use case.

Additionally, new features and bugfixes have been introduced and are **not** documented here. For a comprehensive list of changes, please refer to the changelog.

This release includes many bug fixes and improvements and provides valuable additions to already existing features.

## The *setSdkVersion* method of the *SdkSettings* class has been replaced by *setApplicationVersion*

The `setSdkVersion` method has been deprecated and replaced with `setApplicationVersion`. Update any calls to the old method to use the new method name. The parameter list and meaning remain the same, so this is a simple rename.

The old method is deprecated and will be removed in a future release.

## The *image* getter of the *OverlayItem* class has been replaced by *getImage* method

The `image` getter was deprecated and replaced by the `getImage` method. If your code accessed `OverlayItem.image`, change it to call `OverlayItem.getImage()` or to `OverlayItem.img` instead.

## The *getNavigationRoute* method of the *NavigationService* class now returns *Route?*

The return type of `getNavigationRoute` changed from `Route` to `Route?` to indicate that a navigation route may not always be available. Update your code to handle a nullable return value and add null checks where necessary.

Before:
```dart
Route route = NavigationService.getNavigationRoute();
// Use route
```

After:
```dart
Route? route = NavigationService.getNavigationRoute();

if (route != null) {
	// Do something with route
} else {
	// Handle missing route
}
```

This is a behavioral safety change. Ensure your code does not assume a non-null route.

## The *part* optional parameter of `update` and `setCoordinates` on *Marker* changed from *int?* to *int* (default `0`)

The affected class is `Marker`.
The `part` parameter on `update` and `setCoordinates` previously accepted `int?` with a default of `null`. It now requires an `int` and defaults to `0`.
Update any call sites that pass `null` or omit special handling for `null` to either omit the parameter or pass an explicit integer.

## Camera now animates back into bounds when `zoomLevel` is outside allowed bounds

The affected components are camera handling and map view logic.
When programmatically or by touch setting a `zoomLevel` outside allowed bounds, the camera will now animate back into the valid bounds instead of clamping instantly.

## *SoundMark* and *TextMark* now implement *LogMark*

The affected classes are `SoundMark` and `TextMark`.
Both `SoundMark` and `TextMark` now implement the `LogMark` interface. This change is additive and should not require code changes.

## *LogMetrics* and *RecordMetrics* now implement *Metrics*

The affected classes are `LogMetrics` and `RecordMetrics`.
Both `LogMetrics` and `RecordMetrics` now implement the `Metrics` interface. This is an additive and should not require code changes.

## API adjustments

Some members have been adjusted to be internal, making them no longer accessible as part of the Public API.
Additionally, certain existing members have been deprecated and will now trigger compiler warnings.
Developers should replace the usage of these deprecated members immediately, using the alternative APIs detailed in the changelog or the Public API documentation.

## Structural improvements

Several classes have been relocated to new files to improve overall project organization and maintainability.
If you encounter import errors, please update the import paths in your project to reflect these changes.
Further major restructuring efforts may occur in the near future. Please use the `magiclane_maps_flutter` import as a catch-all to avoid import issues.
