---
description: Documentation for Migrate To 2 19 0
title: Migrate To 2 19 0
---

# Migrate to 2.19.0

This guide outlines the breaking changes introduced in SDK version 2.19.0. Required updates may vary depending on your use case.

Additionally, new features and bugfixes have been introduced and are **not** documented here. For a comprehensive list of changes, please refer to the changelog.

This release adds built-in voices, the timezone service, more sensors related features, many additions to the `GemMapController` and other small improvements.

## The map structure has been changed and is no longer compatible with older SDK versions

A SDK update is required to continue receiving map updates. If a older SDK is used then a callback will be received on the `registerOnWorldwideRoadMapSupportDisabled` callback, indicating the app user that an application update is required:
```dart
SdkSettings.offBoardListener.registerOnWorldwideRoadMapSupportDisabled((Reason reason){
    if (reason == Reason.expiredSDK){
        print("The current SDK version no longer supports worldwide road map data. Please update the app to a version providing worldwide road map data.");
    }
    else if (reason == Reason.noDiskSpace){
        print("Please clear some space on your device.");
    }
});
```

Please update all released and in-development applications to the latest SDK to ensure users have access to the most recent maps and all available online features.

## Dependencies changed in *pubspec.yaml*

The minimum Dart version is now 3.6.0.
The minimum Flutter version is now 3.27.0.
The required `flutter_lints` package is now at version `^5.0.0`.

Executing `flutter clean` and `dart pub upgrade` may be needed.

## Changed parameter type of the *getForecast* method from the *WeatherService* class

The type of the `coords` parameter has been changed from `List<TimeDistanceCoordinate>` to `List<WeatherDurationCoordinates>`.

Before:
```dart
Coordinates qCoords = ...
TimeDistanceCoordinate qTimeCoords = TimeDistanceCoordinate(
    coords: qCoords,
    stamp: Duration(days: 3).inSeconds,
);

final listener = WeatherService.getForecast(
    coords: [qTimeCoords],
    onCompleteCallback: (error, result) {
        // Do something with the result...
    },
);
```

Now:
```dart
Coordinates qCoords = ...
TimeDistanceCoordinate qTimeCoords = WeatherDurationCoordinates(
    coordinates: qCoords,
    duration: Duration(days: 3),
);

final listener = WeatherService.getForecast(
    coords: [qTimeCoords],
    onCompleteCallback: (error, result) {
        // Do something with the result...
    },
);
```

The separation between `TimeDistanceCoordinate` and `WeatherDurationCoordinates` improves clarity.

## The *PtRoute* class related to public transit stop information has been renamed to *PtRouteInfo*

A class with the name `PtRoute` already existed in the SDK and is related to routing.
The class `PtRoute` introduced in 2.18.0 has been renamed to `PtRouteInfo` to avoid confusion.

## Renamed *registerCursorSelectionMapSceneObjectCallback* to *registerCursorSelectionUpdatedMapSceneObjectCallback*

The `registerCursorSelectionMapSceneObjectCallback` method from the `GemMapController` class has been renamed to `registerCursorSelectionUpdatedMapSceneObjectCallback`.

Before:
```dart
GemMapController controller = ...
controller.registerCursorSelectionMapSceneObjectCallback((obj){
    // Do something with the object
})
```

After:
```dart
GemMapController controller = ...
controller.registerCursorSelectionUpdatedMapSceneObjectCallback((obj){
    // Do something with the object
})
```

This change improves consistency in the SDK.

## The *overlayInfo* getter of the *OverlayItem* class is now nullable

The value null is returned when the item has no associated `OverlayInfo` object.

Before:
```dart
OverlayItem item = ...
OverlayInfo info = item.overlayInfo;

if (info.name.isEmpty){
    // The overlay info is not available
}
else {
    // Do something with the overlay info
}
```

After:
```dart
OverlayItem item = ...
OverlayInfo? info = item.overlayInfo;

if (info == null){
    // The overlay info is not available
}
else {
    // Do something with the overlay info
}
```

## The *image* getters of the *OverlayInfo* and *OverlayCategory* are now nullable

The value null is returned when the object has no associated image.
The return type changed from `Uint8List` to `Uint8List?`.

Before:
```dart
OverlayItem item = ...
Uint8List image = item.image;

// No easy way to check if the image is valid as some data is still returned
```

After:
```dart
OverlayItem item = ...
Uint8List? = item.image;

if (image == null){
    // No image available
}
else {
    // Do something with the image
}
```

Same logic applies for `OverlayCategory`s.

## The return type of *getElevationSamples* from *RouteTerrainProfile* has been updated

The method previously returned a loosely typed `Pair<List<dynamic>, double>`. It now returns a properly typed `Pair<List<double>, double>` to ensure type safety and clarity.

Before:
```dart
RouteTerrainProfile profile = ...
Pair<List, double> samples = profile.getElevationSamples(...);
```

After:
```dart
RouteTerrainProfile profile = ...
Pair<List<double>, double> samples = profile.getElevationSamples(...);
```

The method contained `double` in the list, but they were not safety typed. The behavior of the method did not change but casting is now done internally.

## The *isStopped* method from the *DataSource* class has been replaced with a getter

The *isStopped* property is now accessed as a getter instead of a method.
This improves readability and aligns with Dart style for boolean flags.

Before:
```dart
DataSource source = ...
bool stopped = source.isStopped();
```

`

After:
```dart
DataSource source = ...
bool stopped = source.isStopped;
```

## Removed the *osInfo* getter from the *SdkSettings* class

The method provided limited functionality and worked for Android.

The same result can be achieved via the features provided by Flutter and external packages.
See the `Platform.operatingSystemVersion` getter.

## The *deviceModel* getter, setter and constructor parameter of the *RecorderConfiguration* has been deprecated and will be removed

The `hardwareSpecifications` member provides functionality for configuring more features, including the `deviceModel`.

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

The deprecated `deviceModel` uses the config stored inside the `hardwareSpecifications` hashmap.

## The *ExternalPositionData* class and the *positionFromExternalData* method from the *SenseDataFactory* class have been deprecated

The `SenseDataFactory` class provides the `producePosition` method which can be used instead with minimal changes.

Before:
```dart
GemPosition position = SenseDataFactory.positionFromExternalData(
    ExternalPositionData(
        timestamp: DateTime.now().millisecondsSinceEpoch,
        latitude: 36,
        longitude: 40,
        altitude: 10,
        heading: 26,
        speed: 3,
    ),
);
```

After:
```dart
GemPosition position = SenseDataFactory.producePosition(
    acquisitionTime: DateTime.now(),
    satelliteTime: DateTime.now(),
    latitude: 36,
    longitude: 40,
    altitude: 10,
    course: 26,
    speed: 3,
);
```

## The *refreshContentStore* method from the *ContentStore* class has been deprecated and replaced with *refresh*

Use the new *refresh* method for improved clarity and naming consistency.

Before:
```dart
ContentStore store = ...
store.refreshContentStore();
```

After:
```dart
ContentStore store = ...
store.refresh();
```

## The *setActivityRecord* method from the *Recorder* class has been deprecated and replaced with the *activityRecord* setter

The `Recorder` class now uses a property setter for assigning the activity record, improving readability and aligning with Dart conventions.

Before:
```dart
Recorder recorder = ...
recorder.setActivityRecord(record);
```

After:
```dart
Recorder recorder = ...
recorder.activityRecord = record;
```

## The *setTTSLanguage* method from the *SdkSettings* class has been deprecated and replaced with *setTTSVoiceByLanguage*

The name of the method has been changed to be more intuitive and better reflect the role of the method.

Before:
```dart
Language language = ...
SdkSettings settings = ...

settings.setTTSLanguage(language);
```

After:
```dart
Language language = ...
SdkSettings settings = ...

settings.setTTSVoiceByLanguage(language);
```

## New enum values

The `nmeaChunk` value has been added to the `DataType` enum.
The `dr` value has been added to the `FileType` enum.

The ids of the enum values have been changed. Always use the enums and avoid hardcoded int values.
