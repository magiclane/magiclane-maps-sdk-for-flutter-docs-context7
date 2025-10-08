---
description: Documentation for Migrate To 2 15 0
title: Migrate To 2 15 0
---

# Migrate to 2.15.0

This guide outlines the breaking changes introduced in SDK version 2.15.0. Required updates may vary depending on your use case.

Additionally, new features and bugfixes have been introduced and are **not** documented here. For a comprehensive list of changes, please refer to the changelog.

This release brings improvements to the API reference, better error handling and bugfixes.

## Removed methods and properties that were not fully implemented

The `setVoiceByPath` method of the `SdkSettings` class has been removed.
The Flutter SDK does not fully support applying and using voices. In order to change the voice of the TTS instructions please check the documentation provided by the `flutter_tts` package.

The `smallMode` field of the `SignpostImageRenderSettings` class has been removed as the feature is not fully implemented.

## Fixed typos

Many typos were found and fixed:

- the `EPlayingStatus` enum has been renamed to `PlayingStatus`

- the `UnitOfMearsurementAcceleration` enum has been renamed to `UnitOfMeasurementAcceleration`

- the `hoteMotel` value of the `SignpostPictogramType` enum has been changed to `hotelMotel`

- the `insideCityAea` named parameter of the `getOverSpeedThreshold` method provided by the `AlarmService` class has been renamed to `insideCityArea`

## The *logsList* getter from the *RecorderBookmarks* class has been replaced by the *getLogsList*

Before: 
```dart
RecorderBookmarks recorderBookmarks = ...
List<String> logs = recorderBookmarks.logsList;
```

After:
```dart
RecorderBookmarks recorderBookmarks = ...
List<String> logs = recorderBookmarks!.getLogsList();
```

The `getLogsList` method also allows specifying the sorting order.

## Methods were replaced with setters inside the *FollowPositionPreferences* class

The `setTouchHandlerModifyHorizontalAngleLimits` and `setTouchHandlerModifyDistanceLimits` methods from the `FollowPositionPreferences` class have been replaced with `touchHandlerModifyHorizontalAngleLimits` and `touchHandlerModifyDistanceLimits` setters.

Before:
```dart
FollowPositionPreferences preferences = ...
preferences.setTouchHandlerModifyHorizontalAngleLimits(Pair(30.2, 75.0));
preferences.setTouchHandlerModifyDistanceLimits(Pair(30.2, 75.0));
```

After:
```dart
FollowPositionPreferences preferences = ...
preferences.touchHandlerModifyHorizontalAngleLimits = Pair(30.2, 75.0);
preferences.touchHandlerModifyDistanceLimits = Pair(30.2, 75.0);
```

## Methods were replaced with getters inside the *Playback* class

The `getState`, `getDuration`, `getCurrentPosition`, `getSpeedMultiplier`, `getMaxSpeedMultiplier`, `getLogPath`, `getRoute`, `getMinSpeedMultiplier` methods from the `Playback` class were replaced by the `state`, `duration`, `currentPosition`, `speedMultiplier`, `maxSpeedMultiplier`, `logPath`, `route` and `minSpeedMultiplier` getters

Before:
```dart
Playback playback = ...
PlayingStatus status = playback.getState();
```

After:
```dart
Playback playback = ...
PlayingStatus status = playback.state;
```

## Changes regarding content update

### Changes regarding the *update* method

Changed the return type of the `update` method from `GemError` to `ProgressListener?`. Added optional `onCompleteCallback` parameter for listening for the result of the update operation. 
 
The return type of the `update` method is now `ProgressListener?` instead of `GemError`.
If the operation could be **started** then the `update` method returns a non-null `ProgressListener`.
If the operation could not be **started** then the `update` method returns `null` and provides the `GemError` via the `onCompleteCallback` parameter.

The newly added `onCompleteCallback` gets triggered with the result code at the end of the update operation (after `apply` is called) or earlier with the failure error if the update fails before `apply` is called. 

Before:
```dart
ContentUpdater contentUpdater = ...
GemError error = contentUpdater.update(true);
// Do something with the error...
```

After:
```dart
ContentUpdater contentUpdater = ...
ProgressListener listener = contentUpdater.update(
    true,
    onCompleteCallback: (error) {
        // Do something with the error...
    },
),
```

### Changes regarding the *createContentUpdater* method

The `createContentUpdater` now also returns the error code. Replace the return type from `ContentUpdater` to `Pair<ContentUpdater, GemError>`.

- If the operation fails then the `GemError` provided is `GemError.success` and the `ContentUpdater` is valid.

- If the operation succeeds then the `GemError` provided is not `GemError.success` and the `ContentUpdater` is invalid.

If a content updater for the specified type already exists then the error code will be `GemError.exists`

Before:
```dart
ContentUpdater contentUpdater = ContentStore.createContentUpdater(ContentType.roadMap);
// Do something with the content updater...
```

After:
```dart
Pair<ContentUpdater, GemError> result = ContentStore.createContentUpdater(ContentType.roadMap);

if (result.second == GemError.success) {
    ContentUpdater contentUpdater = result.first;
    // Do something with the content updater...
} else {
    print("Creating the content updater failed with error code ${result.second}.");
}
```

### The *updateItem* property of the *ContentStoreItem* class is now nullable

Changed the return type from `ContentStoreItem` to `ContentStoreItem?`. The value `null` is returned when the operation fails (e.g. there is no update in progress for the given item).

Before:
```dart
ContentStoreItem item = ...;
ContentStoreItem updateItem = item.updateItem;
// Do something with the update item
```

After:
```dart
ContentStoreItem item = ...;
ContentStoreItem? updateItem = item.updateItem;
if (updateItem != null){
    // Do something with the update item
} else {
    print("Operation failed");
}
```

## The *step* method from the *Playback* class now returns *void* instead of *GemError*

The `step` method from the `Playback` class has been fixed. It can be used to step through logs containing video recordings frame by frame.
The return type has changed from `GemError` to `void`. In this way the method is now aligned with how it works in the other SDKs provided by Magic Lane.

The error can be retrieved via the `ApiErrorService` class. See the [Usage guidelines](../get-started/usage-guidelines) details for more information.

Before:
```dart
Playback playback = ...
GemError error = playback.step();
```

After:
```dart
Playback playback = ...
playback.step();
```

## The *recorderConfiguration* setter from the *Recorder* class has been replaced with the *setRecorderConfiguration* method

In this way the error code for the operation is also returned.

Before:
```dart
Recorder recorder = ...
RecorderConfiguration config = ...

recorder.recorderConfiguration = recorderConfig;
```

After:
```dart
Recorder recorder = ...
RecorderConfiguration config = ...

GemError error = recorder.setRecorderConfiguration(recorderConfig);
if (error == GemError.success)
    print("Operation succeeded.");
else
    print("Operation failed.");
```

## Some methods from the *Weather* return *TaskHandler?* instead of *TaskHandler*. Result value from the *onCompleteCallback* callback is no longer nullable

Affected methods are `getDailyForecast`, `getHourlyForecast`, `getCurrent`, `getForecast`.
This is similar to the change introduced in the 2.11.0 release regarding `calculateRoute`, `search` and other methods. 

The value `null` is returned when the operation could not be started. When the operation can be started a valid `ProgressListener` is returned.
The `locationForecasts` parameter of the `onCompleteCallback` callback is no longer nullable. The type changed from `List<LocationForecast>?` to `List<LocationForecast>`. An empty list is returned if the operation fails instead of null.

If the operation fails the `onCompleteCallback` callback is called with error and **empty** list of results.
Before this update the `onCompleteCallback` callback was called with error and **null** list of results.
On success the `onCompleteCallback` callback is called with `GemError.success` and non-empty list of results as before.

Before:
```dart
WeatherService.getCurrent(
    coords: [coordinates],
    onCompleteCallback: (GemError error, List<LocationForecast>? result) {
        if (result == null){
            print("Operation failed");
        } else {
            // Perform operations with result...    
        }
    },
);
```

After:
```dart
WeatherService.getCurrent(
    coords: [coordinates],
    onCompleteCallback: (GemError error, List<LocationForecast> result) {
        if (result.isEmpty){
            print("Operation failed");
        } else {
            // Perform operations with result...    
        }
    },
);
```

## Some optional parameters are no longer nullable. Default values have been provided

These are the affected methods and parameters:

- type of the `zoom` parameter of the `createLandmarkStore` method of the `LandmarkStoreService` class from `int?` to `int`. A default value of `-1` has also been provided

- type of the `overwrite` parameter of the `add` method of the `RouteBookmarks` class from `bool?` to `bool`. A default value of `false` has also been provided

- type of the `removeLmkContent` parameter of the `removeCategory` method of the `LandmarkStore` class from `bool?` to `bool`. A default value of `false` has also been provided

If `null` is provided explicitly, it can be safely omitted.

Before:
```dart
final store = LandmarkStoreService.createLandmarkStore('LandmarkName', zoom: null);
```

After:
```dart
final store = LandmarkStoreService.createLandmarkStore('LandmarkName');
```

## Changes to the *RenderSettings* hierarchy. It is now a generic template.

The `RenderSettings` class is now a generic template. The `options` property changed type from `Set<dynamic>` to `Set<T>`.

This influences the following extending classes:

- The type of the `options` property from the `RouteRenderSettings` class changed from `Set<dynamic>` to `Set<RouteRenderOptions>`.

- The type of the `options` property from the `HighlightRenderSettings` class changed from `Set<dynamic>` to `Set<HighlightOptions>`

This change should not usually affect your code unless the polymorphic behavior of the classes is used.
