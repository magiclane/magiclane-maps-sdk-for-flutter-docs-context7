---
description: Documentation for Migrate To 3 0 0
title: Migrate To 3 0 0
---

# Migrate to 3.0.0		

This guide outlines the breaking changes introduced in SDK version 3.0.0. Required updates may vary depending on your use case.

Additionally, new features and bugfixes have been introduced and are **not** documented here. For a comprehensive list of changes, please refer to the changelog.

This release includes a number of deprecations and API renames aimed at unifying naming and simplifying callbacks. The most notable items are the removal of `instance` from `PositionService`, the renaming of the `MapStatus` enum to `ContentStoreStatus`, multiple callback registration method renames on `GemMapController`, and several type and parameter changes described below. The package name has also changed, and the SDK is now distributed via pub.dev.

## Project name changed from *gem_kit* to *magiclane_maps_flutter*

The package name has been changed to `magiclane_maps_flutter`. Update your `pubspec.yaml` file to use the new package name (also see the next section).

A new barrel file `magiclane_maps_flutter.dart` has been added to simplify imports. You can now import all SDK classes from this single file.

Before:
```dart
import 'package:gem_kit/core.dart';
import 'package:gem_kit/map.dart';
import 'package:gem_kit/navigation.dart';
```

After:
```dart
import 'package:magiclane_maps_flutter/magiclane_maps_flutter.dart';
```

This change improves discoverability (as the package is now on pub.dev) and simplifies imports. Also see the next section about package distribution changes.

## The package is now distributed via pub.dev

The SDK is now available on pub.dev. Update your `pubspec.yaml` to use the new package source.

### Pubspec changes

Before:
```yaml
dependencies:
  gem_kit:
    path: plugins/gem_kit
```

After:
```yaml
dependencies:
  magiclane_maps_flutter: ^3.0.0
```

### Android configuration changes

Also update the `maven` block in your `android/build.gradle.kts` file to include the new Maven repository:

Before:
```gradle
maven {
	url = uri("${rootDir}/../plugins/gem_kit/android/build")
}
```

After:
```gradle
maven {
	url = uri("https://developer.magiclane.com/packages/android")
}
```

Make sure to run the following commands to update your dependencies:
```bash
flutter clean
flutter pub get
```

If errors persist on Android after making these changes, try opening the `android` folder in Android Studio and syncing the Gradle files.
No additional code changes are needed for ios, besides updating the `iOS Deployment Target` in your Xcode project settings to at least `14.0`.
See the [Getting Started](../../guides/get-started/integrate-sdk) guide for more details.

## Removed *instance* static property from the *PositionService* class

The `instance` getter was removed. All previous instance methods and properties on `PositionService` were converted to static methods/properties. Call sites that used `PositionService.instance.someMethod()` must call `PositionService.someMethod()` instead.

Before:
```dart
final pos = PositionService.instance;
pos.addPositionListener(...);
```

After:
```dart
PositionService.addPositionListener(...);
```

This change centralizes `PositionService` usage and removes the need to manage an instance.

## Removed many *register....Callback* methods from *GemMapController* (replaced by *registerOn....* variants)

| **Removed Method**                                    | **Replacement Method**                         |
|----------------------------------------------------------|------------------------------------------------|
| `registerTouchHandlerModifyFollowPositionCallback`       | `registerOnTouchHandlerModifyFollowPosition`   |
| `registerMoveCallback`                                   | `registerOnMove`                               |
| `registerLongPressCallback`                              | `registerOnLongPress`                          |
| `registerTwoDoubleTouchesCallback`                       | `registerOnTwoDoubleTouches`                   |
| `registerSwipeCallback`                                  | `registerOnSwipe`                              |
| `registerMapAngleUpdateCallback`                         | `registerOnMapAngleUpdate`                     |
| `registerViewRenderedCallback`                           | `registerOnViewRendered`                       |
| `registerTouchCallback`                                  | `registerOnTouch`                              |
| `registerTwoTouchesCallback`                             | `registerOnTwoTouches`                         |
| `registerPinchSwipeCallback`                             | `registerOnPinchSwipe`                         |
| `registerShoveCallback`                                  | `registerOnShove`                              |
| `registerFollowPositionStateCallback`                    | `registerOnFollowPositionState`                |
| `registerCursorSelectionUpdatedLandmarksCallback`        | `registerOnCursorSelectionUpdatedLandmarks`    |
| `registerCursorSelectionUpdatedMapSceneObjectCallback`   | `registerOnCursorSelectionUpdatedMapSceneObject`|
| `registerCursorSelectionUpdatedRoutesCallback`           | `registerOnCursorSelectionUpdatedRoutes`       |
| `registerCursorSelectionUpdatedMarkersCallback`          | `registerOnCursorSelectionUpdatedMarkers`      |
| `registerHoveredMapLabelHighlightedOverlayItemCallback`  | `registerOnHoveredMapLabelHighlightedOverlayItem` |
| `registerDoubleTouchCallback`                            | `registerOnDoubleTouch`                        |
| `registerCursorSelectionUpdatedTrafficEventsCallback`    | `registerOnCursorSelectionUpdatedTrafficEvents`|
| `registerHoveredMapLabelHighlightedLandmarkCallback`     | `registerOnHoveredMapLabelHighlightedLandmark` |
| `registerRenderMapScaleCallback`                         | `registerOnRenderMapScale`                     |
| `registerMapViewMoveStateChangedCallback`                | `registerOnMapViewMoveStateChanged`            |
| `registerTouchMoveCallback`                              | `registerOnTouchMove`                          |
| `registerTouchPinchCallback`                             | `registerOnTouchPinch`                         |
| `registerCursorSelectionUpdatedOverlayItemsCallback`     | `registerOnCursorSelectionUpdatedOverlayItems` |
| `registerViewportResizedCallback`                        | `registerOnViewportResized`                    |
| `registerCursorSelectionUpdatedPathCallback`             | `registerOnCursorSelectionUpdatedPath`         |
| `registerHoveredMapLabelHighlightedTrafficEventCallback` | `registerOnHoveredMapLabelHighlightedTrafficEvent` |
| `registerSetMapStyleCallback`                            | `registerOnSetMapStyle`                        |
| `registerPinchCallback`                                  | `registerOnPinch`                              |

The old methods were deprecated in version 2.27.0 and have now been removed. Update all call sites to use the new `registerOn...` method names.

## Removed *registerOnProgressCallback* and *registerOnCompleteWithDataCallback* methods from *ProgressListener* and related classes

These were previously deprecated and replaced with `registerOnProgress` and `registerOnCompleteWithData` respectively.

Most API users do not interact with ProgressListener directly, so no code changes are needed in typical cases.

## The *MapStatus* enum renamed to *ContentStoreStatus* and related *status* parameter updates

Affected members are

- Enum: `MapStatus` renamed to `ContentStoreStatus`

- `OffBoardListener.registerOnAvailableContentUpdate(status: ...)`

- `OffBoardListener.registerOnWorldwideRoadMapSupportStatus(status: ...)`

The `MapStatus` enum is now called `ContentStoreStatus`. Any code referring to `MapStatus` must be updated to use `ContentStoreStatus`. The `status` parameter in the two `OffBoardListener` registration methods now uses `ContentStoreStatus`.

This is a rename only; enum values are preserved but the type name changed.

## Data returned via *previewData* moved to *previewDataParameterList* and added new *previewData* property in *OverlayItem*

The previous `previewData` getter was renamed to `previewDataParameterList` to make its semantics explicit. A new `previewData` property was added which has type `OverlayItemParameters?`.

If you accessed the old getter `previewData`, switch to `previewDataParameterList`. It is recommended to use the new `previewData` property instead, as it provides richer structured data.

Before:
```dart
final SearchableParameterList preview = overlayItem.previewData;
```

After (if you want the old list):
```dart
final SearchableParameterList previewList = overlayItem.previewDataParameterList;
```

After (if you want the new structured preview):
```dart
final OverlayItemParameters? preview = overlayItem.previewData;
```

The `OverlayItemParameters` is an abstract class and, depending on the `OverlayItem` type, the actual instance may be one of:

- `SocialReportParameters` for social reports overlay items

- `SafetyParameters` for safety overlay items

- `PublicTransportParameters` for public transport overlay items

- null if the overlay item has a custom type

This change improves type safety and clarity around the preview data associated with overlay items.

## The *data* parameter type changed from *SearchableParameterList* to *TrafficParameters* in *onResult* callbacks provided to *getPreviewData* for *RouteTrafficEvent* and *TrafficEvent* classes

The `data` parameter of the `onResult` callback changed from `SearchableParameterList?` to `TrafficParameters?`. Update callback implementations to handle the new type.

Before:
```dart
routeTrafficEvent.getPreviewData(onResult: (GemError error, SearchableParameterList? data) {
    // handle data
});
```

After:
```dart
routeTrafficEvent.getPreviewData(onResult: (GemError error, TrafficParameters? data) {
    // handle data
});
```

If you previously read `SearchableParameterList` fields directly, migrate to `TrafficParameters` accessors that provide typed access to traffic event details.

## Changed: *contentParameters* type from *SearchableParameterList* to *ContentParameters* on *ContentStoreItem* changed

The property is now nullable and typed as `ContentParameters?`. Update code to null-check and use the new structure.

Before:
```dart
final SearchableParameterList contentParameters = item.contentParameters;
```

After:
```dart
final ContentParameters? contentParameters = item.contentParameters;
if (contentParameters != null) {
    // Do something with contentParameters
}
```

The `ContentParameters` is an abstract class and, depending on the content type, the actual instance may be one of:

- `RoadMapParameters` for road map content

- `VoiceParameters` for voice content (human/computer)

- `StyleParameters` for style content (high quality and low quality)

## The *asyncDownload* callback removed *onProgressCallback* parameter in favor of *onProgress* in *asyncDownload* from *ContentStoreItem*

The deprecated named parameter `onProgressCallback` was removed. Use the new `onProgress` callback parameter instead.

Before:
```dart
item.asyncDownload(onProgressCallback: ...);
```

After:
```dart
item.asyncDownload(onProgress: ...);
```

## The parameters of the *onComplete* callback are now non-nullable in the *ContentStore* class

When the operation finishes with an error, `items` will be an empty list and `isCached` will be `false` instead of `null`.

Before:
```dart
ContentStore.asyncGetStoreContentList(ContentType.roadMap, (GemError err, List<ContentStoreItem>? items, bool? isCached) {
	if (items != null) {
		// handle items
	}
});
```

After:
```dart
ContentStore.asyncGetStoreContentList(ContentType.roadMap, (GemError err, List<ContentStoreItem> items, bool isCached) {
	if (items.isNotEmpty) {
		// handle items
	}
});
```

## Removed EV routing related classes, properties, and conversion methods

The following EV-specific types and conversion helpers were removed:

- `EVRoute` class

- `EVRouteSegment` class

- `EVRouteInstruction` class

- `EVProfile` class

- `Route.toEVRoute()` method

- `RouteSegment.toEVRouteSegment()` method

- `RouteInstruction.toEVRouteInstruction()` method

- `RoutePreferences.evProfile` property

EV related functionality is not available for the time being on the public SDK. These features were not functional in the previous SDK version and have been removed to avoid confusion.
Future SDK releases may reintroduce EV routing features in a revised form.
