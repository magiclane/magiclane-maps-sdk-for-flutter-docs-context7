---
description: Documentation for Migrate To 2 12 0
title: Migrate To 2 12 0
---

# Migrate to 2.12.0

This guide outlines the breaking changes introduced in SDK version 2.12.0. Required updates may vary depending on your use case.

Additionally, new features and bug fixes have been introduced and are **not** documented here. For a comprehensive list of changes, please refer to the changelog.

This release adds important changes related to position, position service, data sources, sense and provides the foundation for adding more sense related features in the future releases.

## *GemPosition* class has been transformed into an interface

The constructor and `toJson`/`fromJson` methods have been removed. In order to instantiate objects use the newly provided `SenseDataFactory` class.

Before:
```dart
GemPosition position = GemPosition();
```

Now:
```dart
GemPosition position = SenseDataFactory.producePosition(); // This method also takes optional parameters
```

## *roadModifiers* and *speedLimit* getters from the *GemPosition* have been moved to the *GemImprovedPosition* interface

A separate `GemImprovedPosition` interface has been added to better differentiate between map-matched and non-map-matched positions. The `roadModifiers` and `speedLimit` getters have been moved to the newly added interface.

Before:
```dart
GemPosition position = ...
Set<RoadModifier> position.roadModifiers;
double limit = position.speedLimit;
```

After:
```dart
GemImprovedPosition improvedPosition = ...
Set<RoadModifier> improvedPosition.roadModifiers;
double limit = improvedPosition.speedLimit;
```

In order to get a `GemImprovedPosition` instance:

- Use the `addImprovedPositionListener` callback from the `PositionService` class

- Use the `getImprovedPosition` method of the `PositionService` class

- Cast a `GemPosition`/`SenseData` variable to a `GemImprovedPosition` if the value of type `GemImprovedPosition` has been upcasted to one of the parent types

- Use the `getLatestData` on a `DataSource` object that supports map-matched positions (the `DataType.improvedPosition` value is found within the list returned by the `availableDataTypes` getter). The value returned by `getLatestData` needs to be upcasted to `GemImprovedPosition`

## The *addImprovedPositionListener* method of the *PositionService* class registers a callback that takes a *GemImprovedPosition* parameter instead of *GemPosition*

This:
```dart
PositionService.instance.addImprovedPositionListener((GemPosition position) {
    // Do operation with GemPosition
});
```

Becomes:
```dart
PositionService.instance.addImprovedPositionListener((GemImprovedPosition position) {
    // Do operation with GemImprovedPosition
});
```

**Note:**
The `GemImprovedPosition` extends `GemPosition` with additional features. Even though using this method with the `GemPosition` type still works (and the value can be later downcasted into a map-matched position), the `GemImprovedPosition` provides additional values such as `roadModifiers` and `speedLimit`.

## Deprecated *timestamp* getter from the *GemPosition* interface

As an additional time value has been added to the `GemPosition` interface, the old `timestamp` has been deprecated and the `satelliteTime` was added to better reflect the meaning of the value.

Before:
```dart
GemPosition position = ...
DateTime timestamp = position.timestamp;
```

Now:
```dart
GemPosition position = ...
DateTime timestamp = position.satelliteTime;
```

The old `timestamp` getter will be removed in a future release.

## The *pushData* method of the *DataSource* class now takes a single required *SenseData* parameter instead of optional *ExternalAccelerationData* and *ExternalPositionData*

The `pushData` method no longer accepts `ExternalPositionData` values as parameters. These can be converted into `GemPosition` (which extend `SenseData`) using the newly added `SenseDataFactory` class.

This:
```dart
ExternalPositionData externalPosition = ExternalPositionData(
    timestamp: DateTime.now().millisecondsSinceEpoch,
    latitude: 36,
    longitude: 40,
    altitude: 10,
    heading: 26,
    speed: 3,
);

dataSource.pushData(positionData: externalPosition);
```

Becomes:
```dart
ExternalPositionData externalPosition = ExternalPositionData(
    timestamp: DateTime.now().millisecondsSinceEpoch,
    latitude: 36,
    longitude: 40,
    altitude: 10,
    heading: 26,
    speed: 3,
);

dataSource.pushData(SenseDataFactory.positionFromExternalData(externalPosition));
```

## The *startRecording* method of the *Recorder* class is now async and needs to be awaited

Not awaiting the `startRecording` method might lead into unexpected and unpredictable behaviour.

This:
```dart
Recorder recorder = Recorder.create(recorderConfig);
GemError error = recorder.startRecording();
```

Becomes:
```dart
Recorder recorder = Recorder.create(recorderConfig);
GemError error = await recorder.startRecording();
```


