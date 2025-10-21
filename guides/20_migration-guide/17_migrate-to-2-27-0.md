---
description: Documentation for Migrate To 2 27 0
title: Migrate To 2 27 0
---

# Migrate to 2.27.0

This guide outlines the breaking changes introduced in SDK version 2.27.0. Required updates may vary depending on your use case.

Additionally, new features and bugfixes have been introduced and are **not** documented here. For a comprehensive list of changes, please refer to the changelog.

This release removes previously deprecated types and members, refactors listener and registration method names, improves API consistency. The sections below document the removals and the API changes that may require you to update your code.

This release removes all deprecated members and types, introducing several breaking changes.
Depending on how you use the SDK, upgrading to this version may require substantial code updates.

These changes are part of our ongoing effort to improve the consistency and usability while retiring legacy code that has been replaced with more robust alternatives.

With plans for a future pub.dev release underway, this update represents a significant milestone in that process.

## NavigationService *startSimulation*/*startNavigation* signature changes

The affected members are: `NavigationService.startSimulation` and `NavigationService.startNavigation`

The following changes were made to the signatures of these methods:

- `onNavigationInstructionUpdate` positional callback parameter was removed. The `NavigationEventType` enum was also removed.

- `autoPlaySound` parameter was removed; use `SoundPlayingService` to toggle TTS playback

Before:
```dart
TaskHandler taskHandler = NavigationService.startNavigation(
    routes.first,
    (eventType, instruction) { // <- This is the previously deprecated callback that was removed
        if (eventType == NavigationEventType.navigationInstructionUpdate) {
            // Do operation with instruction
        }
        if (eventType == NavigationEventType.destinationReached) {
            // Do operation when destination is reached
        }
        if (eventType == NavigationEventType.error) {
            // Do operation with error is triggered
        }
    },
    autoPlaySound: true, // <- This parameter was removed
);
```

After:
```dart
// The sound playing service can be used to control TTS playback at any time
SoundPlayingService.canPlaySounds = true;

TaskHandler? taskHandler = NavigationService.startNavigation(
    routes.first,
    //highlight-start
    onNavigationInstruction: (NavigationInstruction instruction, Set<NavigationInstructionUpdateEvents> events) {
        // Do operation with instruction
        // Note: details about reasons why a new instruction is triggered are now available in events
    },
    onDestinationReached: (Landmark landmark) {
        // Do operation when destination is reached
        // Note: the destination is also provided as a landmark 
    },
    onError: (GemError error) {
        // Do operation with error
        // Note: The error is also provided
    },
    //highlight-end
);
```

Remove the `onNavigationInstructionUpdate` callback and replace it with the new specialized callbacks. Remove the `autoPlaySound` parameter and use `SoundPlayingService` to control TTS playback.

This change continues the deprecation added in the 2.11.0 release, replacing the `onNavigationInstructionUpdate` with more specialized callbacks. Check the [Migrate to 2.21.0](/guides/migration-guide/migrate-to-2-11-0) guide for more details.

## Removed the *ExternalPositionData* class and *positionFromExternalData* method from *SenseDataFactory* class

Both were previously deprecated and are now removed. Use `SenseDataFactory.producePosition` instead to create positions for creating instances.
This change continues the deprecation added in the 2.12.0 release. Check the [Migrate to 2.12.0](/guides/migration-guide/migrate-to-2-12-0) guide for more details.

Before:
```dart
ExternalPositionData data = ...;
GemPosition pos = SenseDataFactory.positionFromExternalData(data);
```

After:
```dart
final pos = SenseDataFactory.producePosition(
	...
);
```

Note: `producePosition` has an updated default `provider` value (see below).

## Default provider change in *SenseDataFactory.producePosition*

Affected members: `SenseDataFactory.producePosition` default `provider` changed from `Provider.unknown` to `Provider.gps`.

If you relied on the old default provider, explicitly pass `provider: Provider.unknown` when calling `producePosition` to keep previous behavior.

This change makes it easier to create GPS-based positions which can be passed to other methods without needing to specify the provider explicitly.

## Removed deprecated names for angles

These deprecated properties were removed. Equivalent functionality is available through other properties/parameters.

The following properties can be used instead:

| Removed Property        | Replacement Property/Method               | Affected classes and methods               |
|------------------------|--------------------------------------------|--------------------------------------------|
| `headingInDegrees`     | `mapAngle`                                 | `MapView` class                            |
| `pitchInDegrees`       | `viewAngle`                                | `MapView` class                            |
| `angle`                | `mapAngle`                                 | `FollowPositionPreferences.setMapRotationMode` method |
| `rotationAngle`        | `mapAngle`                                 | `FollowPositionPreferences` and `MapViewPreferences` classes |

This continues the deprecation added in the 2.24.0 release. Check the [Migrate to 2.24.0](/guides/migration-guide/migrate-to-2-24-0) guide for more details.
This change improves API consistency by using the angles names consistently across the SDK.

## Removed *isEmpty* getter in geographic area classes

Affected members: `GeographicArea.isEmpty`, `TilesCollectionGeographicArea.isEmpty`, `RectangleGeographicArea.isEmpty`, `PolygonGeographicArea.isEmpty`, `CircleGeographicArea.isEmpty`

The `isEmpty` getter was removed in favor of `isDefault`. Replace `isEmpty` with `isDefault`.

Before:
```dart
if (area.isEmpty) { ... }
```

After:
```dart
if (area.isDefault) { ... }
```

The `isEmpty` getter was deprecated before this release.

## Removed *timestamp* from *GemPosition*

Affected members: `GemPosition.timestamp` and `GemImprovedPosition.timestamp`
The `timestamp` property was deprecated and removed. Use `acquisitionTime` instead.

Before:
```dart
DateTime timestamp = gemPosition.timestamp;
```

After:
```dart
DateTime timestamp = gemPosition.acquisitionTime;
```

## Replaced *getImprovedPosition*/*getPosition* methods from *PositionService* with getters

Affected members are `PositionService.getImprovedPosition` and `PositionService.getPosition`. They were previously deprecated and are now removed.
Use `PositionService.improvedPosition`, `PositionService.position` getters instead.

Before:
```dart
GemPosition? position = PositionService.instance.getPosition();
```

After:
```dart
GemPosition? position = PositionService.instance.position;
```

## Deprecated methods in *SdkSettings* were removed

Affected members are `SdkSettings.setTTSLanguage` and `SdkSettings.setAllowConnection`.
Use `SdkSettings.setTTSVoiceByLanguage` and `SdkSettings.setAllowInternetConnection` instead.

Rename your calls accordingly.

Before:
```dart
Language language = ...

sdkSettings.setTTSLanguage(language);
sdkSettings.setAllowConnection(true);
```

After:
```dart
Language language = ...

sdkSettings.setTTSVoiceByLanguage(language);
sdkSettings.setAllowInternetConnection(true);
```

The `setAllowConnection` also provided registration for various callbacks. These are now registered separately on `OffBoardListener` class.
Obtain an `OffBoardListener` instance from `SdkSettings.offBoardListener` and register your callbacks there:
```dart
SdkSettings.offBoardListener.registerOnConnectionStatusUpdated((isConnected){
    // Handle connection status updates
});
```

This change continues the deprecation introduced in the 2.18.0 release. Check the [Migrate to 2.18.0](/guides/migration-guide/migrate-to-2-18-0) guide for more details.

## Removed *horizontalaccuracy* and *verticalaccuracy* from *Coordinates*

Affected members: `Coordinates.horizontalaccuracy`, `Coordinates.verticalaccuracy`

These accuracy fields were deprecated and removed. No direct replacement is provided.
Horizontal and vertical accuracy are available on the `GemPosition` and `GemImprovedPosition` classes.

## Moved *deviceModel* into *hardwareSpecifications* on *RecorderConfiguration*

Set the device model into `hardwareSpecifications` instead of the removed `deviceModel` field.

Before:
```
RecorderConfiguration config = RecorderConfiguration(
    deviceModel: "iPhone 14 Pro",
    // Other settings...
)
```

After:
```dart
RecorderConfiguration config = RecorderConfiguration(
    hardwareSpecifications: {
        HardwareSpecification.deviceModel: "iPhone 14 Pro",
    },
    // Other settings...
)
```

This change continues the deprecation added in the 2.19.0 release. Check the [Migrate to 2.19.0](/guides/migration-guide/migrate-to-2-19-0) guide for more details.

## The *refreshContentStore* method has been renamed to *refresh* on *ContentStore*

Replace the previously deprecated `refreshContentStore` method with the new `refresh` method.

Before:
```dart
ContentStore.refreshContentStore();
```

After:
```dart
ContentStore.refresh();
```

## The *setActivityRecord* methods replaced by *activityRecord* setter on *Recorder*

Replace calls to the deprecated `setActivityRecord` method with the new `activityRecord` setter.

Before:
```dart
recorder.setActivityRecord(record);
```

After:
```dart
recorder.activityRecord = record;
```

## Removed *setNorthFixedFlag* from *MapViewPreferences*

The deprecated method was removed. Use the `northFixedFlag` property instead.

Before:
```dart
mapViewPreferences.setNorthFixedFlag(true);
```

After:
```dart
mapViewPreferences.northFixedFlag = true;
```

## Removed many getters from *ExternalInfo* in favor of property getters

Affected members: `ExternalInfo.getWikiPageTitle`, `getWikiImagesCount`, `getWikiPageDescription`, `getWikiPageUrl`, `getWikiPageLanguage`

| Removed Method                 | Replacement Property        |
|--------------------------------|-----------------------------|
| `getWikiPageTitle`             | `wikiPageTitle`             |
| `getWikiImagesCount`           | `imagesCount`               |
| `getWikiPageDescription`       | `wikiPageDescription`       |
| `getWikiPageUrl`               | `wikiPageUrl`               |
| `getWikiPageLanguage`          | `wikiPageLanguage`          |

Before:
```dart
final title = externalInfo.getWikiPageTitle();
```

After:
```dart
final title = externalInfo.wikiPageTitle;
```

## Renamed *IGemPositionListener* to *GemPositionListener*

The public listener `IGemPositionListener` was renamed to `GemPositionListener`.
The previous `GemPositionListener` class is now `GemPositionListenerImpl` (not exposed in the public API but is used internally).

Additionally, `addPositionListener` and `addImprovedPositionListener` now return an object whose class is the new `GemPositionListener`. The `removeListener` method now expects a `GemPositionListener`.

A simple find-and-replace of `IGemPositionListener` to `GemPositionListener` should suffice in most cases.

## Deprecated *registerOnProgressCallback* and *registerOnCompleteWithDataCallback* methods in *ProgressListener* and *EventDrivenProgressListener* classes

Affected methods are `registerOnProgressCallback` and `registerOnCompleteWithDataCallback`.
The new methods are `registerOnProgress` and `registerOnCompleteWithData`.

The API user should not be affected as the `ProgressListener` and the `EventDrivenProgressListener` classes are usually not used directly.

## All *register...Callback* methods on *GemMapController* renamed to *registerOn....*

The old names were deprecated and replaced with the new names. The new names are more consistent with other listener registration methods in the SDK.

The old pattern was `register...Callback` and the new pattern is `registerOn...`.

| **Deprecated Method**                                    | **Replacement Method**                         |
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

The old methods are now deprecated and will be removed in a **very soon** future release. It is recommended to update your code to use the new methods.

## NavigationInstruction next/nextNext/previous properties are now nullable

Affected members: `NavigationInstruction.nextNextInstruction`, `NavigationInstruction.previousInstruction`, `NavigationInstruction.nextInstruction`. The types changed from `RouteInstruction` to `RouteInstruction?`

Before:
```dart
RouteInstruction? instruction = navigationInstruction.nextInstruction;
```

After:
```dart
RouteInstruction instruction = navigationInstruction.nextInstruction;
```

## Removed deprecated callback parameters: use *onComplete* and new names

Affected members: multiple methods where `onCompleteCallback` was removed and replaced by `onComplete`.

The `onCompleteCallback` parameters for several async-style methods were removed. Use the replacement `onComplete` parameter which receives the same arguments. Replace named parameter names as needed.

The methods affected are:

- `ProjectionService.convert`

- `TimezoneService.getTimezoneInfoFromTimezoneId`, `TimezoneService.getTimezoneInfoFromCoordinates`

- `Weather.getHourlyForecast` `Weather.getCurrent` `Weather.getDailyForecast` `Weather.getForecast`,

- `LandmarkStore.importLandmarksWithDataBuffer`

- `ContentStore.asyncGetStoreFilteredList`

- `ContentUpdater.update`

## Changed positional parameter: *getCountryFlagImgByIndex* now takes positional index

Affected members: `MapDetails.getCountryFlagImgByIndex`

The `index` parameter is now positional instead of named. Update call site accordingly.

Before:
```dart
MapDetails.getCountryFlagImgByIndex(index: 2);
```

After:
```dart
MapDetails.getCountryFlagImgByIndex(2);
```

## *RectType\<T\>* replaced with *Rectangle*

Affected members: many `GemView`/`GemMapController` methods and properties that used `RectType<T>` now use `Rectangle<T>`.

Update all method calls and type annotations from `RectType<T>` to `Rectangle<T>`.

Complete list of affected API changes:

- **GemView** and **GemMapController** classes

    - **Method parameters**

        - `centerOnRoute`

        - `getOptimalRoutesCenterViewport`

        - `getOptimalHighlightCenterViewport`

        - `transformScreenToWgsRect`

        - `centerOnRoutePart`

        - `checkObjectVisibility`

        - `centerOnAreaRect`

        - `centerOnRoutes`

        - `centerOnMapRoutes`

        - `setClippingArea`

        - `getVisibleRouteInterval`

    - **Method return types**

        - `getOptimalRoutesCenterViewport`

        - `getOptimalHighlightCenterViewport`

    - **Getters**

        - `viewportF`

        - `viewport`

- **MapSceneObject** class

    - **Method return type**

        - `getScreenRect`

- **MapViewPreferences** class

    - **Properties**

        - `focusViewport`

        - `mapScalePosition`

A simple find-and-replace of `RectType<` to `Rectangle<` should suffice in most cases.

## Enum updates

- Removed deprecated `me` value from the `RoutePathAlgorithm` enum — use `ml` instead. 

- Removed deprecated `downloadWaiting` value from `ContentStoreItemStatus` — use the more specific `waiting...` values. The numeric values of the enum were adjusted.

- Added `geofence`, `overlays` values to `ContentStoreItemStatus`

If your code branches on enum values, update `switch` statements and add cases for the new values where appropriate.

Always use the enum value names instead of numeric values to avoid issues when enum values are added or removed.

## Removed unused classes: *MapViewOverlayCollection* and *MarkerCustomRenderData*

Affected members: `MapViewOverlayCollection`, `MarkerCustomRenderData`

The `overlays` property on `MapViewPreferences` was also removed as it referenced the removed `MapViewOverlayCollection` class.

These classes were unused and removed. Remove references to them from your code. There is no replacement as they had limited functionality.

## Exception *json* properties changed to *Map* from *String*

Affected members: `MapDisposedException.json`, `ObjectNotAliveException.json` (type changed from `String` to `Map<String, dynamic>`)

These changes should not affect most users as these exceptions are rarely caught directly.
