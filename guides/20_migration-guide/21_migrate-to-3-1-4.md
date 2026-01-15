---
description: Documentation for Migrate To 3 1 4
title: Migrate To 3 1 4
---

# Migrate to 3.1.4

This guide outlines the breaking changes introduced in SDK version 3.1.4. Required updates may vary depending on your use case.

Additionally, new features and bugfixes have been introduced and are **not** documented here. For a comprehensive list of changes, please refer to the changelog.

This release includes many bug fixes and standardizations to improve consistency across the SDK. Follow the steps below to keep your project compatible with 3.1.4.

## The *create* methods from the *ParameterList* and *SearchableParameterList* classes have been removed

Use the provided constructors instead.

Before:
```dart
ParameterList params = ParameterList.create();
```

After:
```dart
ParameterList params = ParameterList();
```

## The *notifyOnNewImprovedPosition* and *notifyOnNewPosition* methods from the *GemPositionListener* class have been removed

These methods were intended for internal use only and are no longer accessible.

##  The *routeResultType* setter from the *RoutePreferences* class has been removed

The `routeResultType` property is now read-only to better reflect its intended usage.

## The *roundTripParameters* getter from the *RoutePreferences* class has been removed

Use the other round trip related properties instead.

## The *setClippingArea* method from the *GemMapController* class has been deprecated and will be removed in a future release

The method does not function as intended and is not relevant for UI applications built with Flutter.
Use the Flutter-included widgets for resizing and clipping instead.

## Many methods are now internal and cannot be accessed directly

Examples of methods no longer accessible include:

- the `on...` methods from various listener classes

- the `fromJson` and `toJson` methods used for serialization and deserialization. The expected JSON structure is not part of the public API and may change without notice.

- the `create` methods from various classes. Use the provided constructors instead.

This change improves encapsulation and reduces the public API surface area. It also ensures that the API contract is clear and stable.

## Multiple methods are now property getters and setters

The following methods have been converted to properties for improved consistency and usability:

| Class                 | Previous method(s)                                                                                               | New property/properties                                                                                 |
|-----------------------|------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------|
| MarkerMatch           | `getMarker`                                                                                                      | `marker`                                                                                                  |
| MarkerInfo            | `getCoords`                                                                                                      | `coords`                                                                                                  |
| ContentStore          | `getStoreFilteredList`                                                                                           | `storeFilteredList`                                                                                       |
| LandmarkStore         | `getFilePath`                                                                                                    | `filePath`                                                                                                 |
| MapDetails            | `getMapProviderIds`, `getCountryDataCount`, `getMapReleaseInfo`                                                  | `mapProviderIds`, `countryDataCount`, `mapReleaseInfo`                                                    |
| MapDownloaderService  | `getMaxSquareKm`, `setMaxSquareKm`                                                                               | `maxSquareKm`                                                                                             |
| SdkSettings           | `getVoice`                                                                                                       | `voice`                                                                                                   |
| DriverBehaviour       | `getOngoingAnalysis`, `getAllDriverBehaviourAnalyses`, `getInstantaneousScores`, `getLastAnalysis`               | `ongoingAnalysis`, `allDriverBehaviourAnalyses`, `instantaneousScores`, `lastAnalysis`                    |
| MappedDrivingEvent    | `getTimestamp`, `getCoordinates`                                                                                 | `timestamp`, `coordinates`                                                                                |
| Path                  | `getLandmarkList`                                                                                                | `landmarkList`                                                                                            |
| TimezoneService       | `getTimezoneInfoTimezoneIdSync`                                                                                  | `getTimezoneInfoFromTimezoneIdSync` (renamed)                                                             |
| Debug                 | `getUsedMemory`, `getTotalMemory`, `getFreeMemory`, `getMaxUsedMemory`, `getAndroidVersion`, `getAppIOInfo`, `getStyleBuilderUrls`, `getRoutingAlgoModifiers`, `getNavigationModifiers`, `timeToBetterRouteSec`, `getServicesIds`, `getAllWeatherConditions`, `isMainThread`, `getMapViewMaxZoomRanges`, `isRawPositionTrackerEnabled`, `getSdkLogDumpPath` | `usedMemory`, `totalMemory`, `freeMemory`, `maxUsedMemory`, `androidVersion`, `appIOInfo`, `styleBuilderUrls`, `routingAlgoModifiers`, `navigationModifiers`, `timeToBetterRoute`, `servicesIds`, `allWeatherConditions`, `mainThread`, `mapViewMaxZoomRanges`, `rawPositionTrackerEnabled`, `sdkLogDumpPath` |
| MapSceneObject        | `getDefPositionTrackerAccuracyCircleColor`                                                                       | `defPositionTrackerAccuracyCircleColor`                                                                   |
| RoutePreferences      | `getRoundTripRange`, `getRoundTripRangeType`, `getRoundTripRandomSeed`                                          | `roundTripRange`, `roundTripRangeType`, `roundTripRandomSeed`                                            |

This change improves code readability and aligns with Dart conventions.

## Changes to the *RoutePreferences* class

The behavior of certain properties in the `RoutePreferences` class has been modified and additional checks have been implemented. Setting some properties may invalidate other fields.
We recommend reviewing your usage of the `RoutePreferences` class to ensure compatibility with these changes.

## Some methods taking/returning a *RectangleGeographicArea* parameter/value have been updated to accept a *GeographicArea* instead

The affected methods include:

- the `area` property from the `TilesCollectionGeographicArea` class

- the `centerOnAreaRect` method from the `GemView` class (accepts a `GeographicArea` parameter)

- the `centerOnArea` and `centerOnAreaRect` methods from the `GemMapController` class (accept a `GeographicArea` parameter)

- the `locationHint` parameter from the `search` and `searchInArea` methods from the `SearchService` class (accepts a `GeographicArea?` parameter)

This change enhances flexibility by allowing the use of various geographic area types.

## The *EventHandler* type has been replaced with *ProgressListener*

A simple find and replace should be sufficient to update your code. The affected methods are:

- the `report`, `confirmReport`, `deleteReport`, `updateReport`, `denyReport`, `addComment`, and `cancel` methods from the `SocialOverlay` class

- the `addListener` and `removeListener` methods from the `Recorder` class

- the `requestWikiInfo` and `cancelWikiInfo` methods from the `ExternalInfoService` class

This change improves type safety and clarity by using specific listener interfaces.

## The *persistentRoadblockListener* property from the *TrafficService* has nullable type now

The type of the `persistentRoadblockListener` property from the `TrafficService` class changed from `PersistentRoadblockListener` to `PersistentRoadblockListener?`.
This change allows for unregistering the listener by setting the property to `null`.

## The *setAllowInternetConnection* method from the *SdkSettings* class is now *Future* and must be awaited

The `setAllowInternetConnection` method from the `SdkSettings` class has been updated to return a `Future<void>`.
You must now await this method to ensure that the operation completes before proceeding.

This change was required to fix a bug where internal verification of internet connectivity settings would override the requested setting if not awaited.

## Some properties from the *RoutePreferences* class have been made non-nullable

The following properties from the `RoutePreferences` class are now non-nullable:

- `bikeProfile` (type changed from `BikeProfileElectricBikeProfile?` to `BikeProfileElectricBikeProfile`)

- `truckProfile` (type changed from `TruckProfile?` to `TruckProfile`)

- `carProfile` (type changed from `CarProfile?` to `CarProfile`)

- `roundTripParameters` (type changed from `RoundTripParameters?` to `RoundTripParameters`)

## The *language* property from the *ContentStoreItem* class has been made nullable to better reflect its optional nature

The type of the `language` property from the `ContentStoreItem` class changed from `Language` to `Language?`.

Before this change, if a `ContentStoreItem` did not have a language specified, it would default to a `Language` with empty values.
Now, the `language` property will be `null` if no language is specified, allowing for clearer handling of optional language data.

## Many register methods now accept nullable callback parameters for unregistering listeners

The affected methods include:

- the `registerOnVolumeChangedByKeys` method from the `SoundPlayingListener` class

- the `registerOnPlayingStatusChanged` method from the `DataSourceListener` class

- the `registerOnProgressChanged` method from the `DataSourceListener` class

- the `registerOnNewData` method from the `DataSourceListener` class

- the `registerOnDataInterruptionEvent` method from the `DataSourceListener` class

This change allows for unregistering listeners by passing `null` as the callback parameter.

## The type of the *getPreviewExtendedData* method from the *OverlayItem* class has been changed to nullable

The return type of the `getPreviewExtendedData` method from the `OverlayItem` class changed from `OverlayItemPreviewExtendedData` to `OverlayItemPreviewExtendedData?`.
This change reflects that the method may return `null` if the operation to retrieve the preview extended data is unsuccessful.

## The *onComplete* parameter type of the *asyncGetStoreFilteredList* method from the *ContentStore* class has been changed

Before this change, the `onComplete` parameter type of the `asyncGetStoreFilteredList` method from the `ContentStore` provided a nullable list of `ContentStoreItem` objects.
Now, the `onComplete` parameter type has been changed to provide a non-nullable list of `ContentStoreItem` objects (`List<ContentStoreItem>`).

The provided list is now empty when the operation fails.

Before:
```dart
ContentStore.asyncGetStoreFilteredList(
    ...,
    onComplete: (GemError err, List<ContentStoreItem>? items) {
        if (err == GemError.success && items != null) {
            // Handle items
        }
    }
)
```

After:
```dart
ContentStore.asyncGetStoreFilteredList(
    ...,
    onComplete: (GemError err, List<ContentStoreItem> items) {
        if (err == GemError.success) {
            // Handle items
        }
    }
)
```

## The *playText* method from the *SoundPlayingService* class

Before:
```dart
SoundPlayingService.playText("Magic Lane");
```

After:
```dart
await SoundPlayingService.playText("Magic Lane");
```

This change fixes a crash when working with Android Auto.

## Changes to the *ExternalImageQuality* values

The `ExternalImageQuality` numeric id values have been updated to follow the changes made in the Wikipedia API.
Always use the enum values instead of hardcoding the numeric ids to ensure compatibility with future changes.

## Changes to the *RoadInfo* class

The `RoadInfo` class no longer includes public constructor or setters.
Instances of the `RoadInfo` class can only be obtained through methods provided by the SDK.

This change improves encapsulation and ensures that `RoadInfo` instances are created and managed correctly by the SDK and
fixes issues with the `getRoadInfoImg` method from the `NavigationInstruction` class.

## The *getCollectionAt* method from the *MapViewMarkerCollections* class has changed return type to nullable

The return type of the `getCollectionAt` method from the `MapViewMarkerCollections` class changed from `MarkerCollection` to `MarkerCollection?`.
This change reflects that the method may return `null` if the specified index is out of bounds.

Before:
```dart
MarkerCollection collection = mapViewMarkerCollections.getCollectionAt(index);
// Use collection
```

After:
```dart
MarkerCollection? collection = mapViewMarkerCollections.getCollectionAt(index);
if (collection != null) {
    // Use collection
}
```

This change improves safety by requiring null checks when accessing collections by index.
