---
description: Documentation for Sensors And Data Sources
title: Sensors And Data Sources
---

# Sensors and data sources

This section provides an overview of how the Maps Flutter SDK integrates with various sensors and external data sources to enhance map functionality and interactivity. From GPS and compass data to accelerometer readings and custom telemetry inputs, the SDK is designed to support a wide range of sensor-driven scenarios. 

You'll learn how to access and configure these inputs, how the SDK responds to real-time changes, and how to incorporate your own data streams into the mapping experience. Whether you're building navigation apps, augmented reality layers, or location-aware services, this section will guide you through the sensor and data integration process.

## Sensor types

The supported sensor data types can be summarized in the following table:

| **Type**              | **Description** |
|-----------------------|-----------------|
| **Acceleration**      | Measures linear movement of the device in three-dimensional space. Useful for detecting motion, steps, or sudden changes in speed. |
| **Activity**          | Represents user activity such as walking, running, or being stationary, typically inferred from motion data. Only available on Android devices.  |
| **Attitude**          | Describes the orientation of the device in 3D space, often expressed as Euler angles or quaternions. |
| **Battery**           | Provides battery status information such as charge level and power state. |
| **Camera**            | Indicates data coming from or triggered by the device's camera, such as frames or detection events. |
| **Compass**           | Gives directional heading relative to magnetic or true north using magnetometer data. |
| **Magnetic Field**    | Reports raw magnetic field strength, useful for environmental sensing or heading correction. |
| **Orientation**       | Combines multiple sensors (like accelerometer and magnetometer) to calculate absolute device orientation. |
| **Position**          | Basic geographic position data, including latitude, longitude, and optionally altitude. |
| **Improved Position** | Enhanced position data that has been refined using filtering, correction services, or sensor fusion. |
| **Gyroscope**         | Measures the rate of rotation around the device’s axes, used to detect turns and angular movement. |
| **Temperature**       | Provides temperature readings, either ambient or internal device temperature. |
| **Notification**      | Represents external or system-level events that are not tied to physical sensors. |
| **Mount Information** | Describes how the device is physically mounted or oriented within a fixed system, such as in a vehicle. |
| **Heart Rate**        | Biometric data representing beats per minute, typically from a fitness or health sensor. |
| **NMEA Chunk**        | Raw navigation data in NMEA sentence format, typically from GNSS receivers for high-precision tracking. Only available on Android devices. |
| **Unknown**           | A fallback type used when the source of the data cannot be determined. |

More details about the `Position` and `ImprovedPosition` classes are available [here](../core/positions).

When using DataType values, ensure that the specific types are supported on the target platform.
Attempting to create data sources or recordings with unsupported types may result in failures.

## Working with data sources

A simplified view of the main classes used to work with data sources can be seen in the following diagram:

There are multiple possible data types, represented by the `DataType` enum. Each sensor value is stored in a class that is derived from `SenseData`. Two such classes are `GemPosition` and `Acceleration`.

If you want to create objects of these types, a helper class `SenseDataFactory` is provided. This class has static methods like `producePosition`, `produceAcceleration` that can create custom sensor data. In principle, this will only be necessary if you want to create a custom data source that will be fed with custom data.

You can create a `DataSource` by using one of the static methods:

- `createLiveDataSource`: Creates a data source that collects data from the device’s built-in sensors in real time. This is the most common use case for applications relying on actual sensor input.

- `createExternalDataSource`: Creates a custom data source that accepts user-supplied data. You can feed data into this source via the `pushData` method. Note that `pushData` will return `false` if used with a non-external source.

- `createLogDataSource`: Creates a data source that replays data from a previously recorded session (log file: gpx, nmea). This is useful for debugging, training, or offline data processing. See the [Recorder docs](./recorder) for information about recording data.

- `createSimulationDataSource`: Creates a data source that simulates movement along a specified route. It can be used for UI prototyping, testing, or feature validation without relying on real-world movement.

The first two types (live and external) are categorized under `DataSourceType.live`, whereas the latter two (log and simulation) fall under `DataSourceType.playback`.

By default, a data source starts automatically upon creation. However, it's possible that it hasn't fully initialized by the time you obtain the data source object.

If you add a `DataSourceListener` immediately after acquiring the data source, there's a chance you'll miss the initial "playing status changed" notification that indicates the data source has started—since it may already be in the started state when the listener is attached.

### Configuring and Controlling a Data Source

Once created, a data source can be stopped or started using the appropriate control methods:
```dart
dataSource.stop();
// ...
dataSource.start();
```

You can also configure a data source’s behavior using methods like:

- `setConfiguration`: to set the sampling rate or data filtering behavior.

- `setMockPosition`: to simulate location updates.

The `setMockPosition` method is only available for live data sources and supports only the `DataType.position` type.
To mock other data types, use an external `DataSource`.

### Using `DataSourceListener`

To receive updates from a data source, you can register a `DataSourceListener`. This listener allows you to react to various events such as:

- Changes in the playing status of the data source.

- Interruptions in data flow (e.g., sensor stopped, app went to background, etc.).

- New sensor data becoming available.

- Progress updates during playback.

You can create a listener using the factory constructor and pass the appropriate callbacks:
```dart
final listener = DataSourceListener(
  onPlayingStatusChanged: (dataType, status) {
    print('Status for $dataType changed to $status');
  },
  onDataInterruptionEvent: (dataType, reason, ended) {
    print('Data interruption on $dataType: $reason. Ended: $ended');
  },
  onNewData: (data) {
    print('New data received: $data');
  },
  onProgressChanged: (progress) {
    print('Playback progress: $progress%');
  },
);
```

Once created, this listener can be registered with a `DataSource`, for a specific `DataType` (in this case the position):
```dart
myDataSource.addListener(DataType.position, listener);
```

Later, you can remove the listener when it’s no longer needed:
```dart
myDataSource.removeListener(DataType.position, listener);
```

## Using the `Playback` interface

The `Playback` interface allows you to control data sources that support playback functionality—specifically those of type `DataSourceType.playback`, such as *log files* or *simulated route replays*. **It is not compatible with live or custom data sources**.

To access a `Playback` instance, you can check the type of the data source and retrieve it accordingly:
```dart
if(myDataSource.dataSourceType == DataSourceType.playback) {
  final playback = myDataSource.playback!;

  playback.pause();
  // ...
  playback.resume();
}
```

As shown above, playback-enabled data sources can be paused and resumed. Additionally, you can adjust the playback speed by setting a `speedMultiplier`, which must fall within the range defined by `Playback.minSpeedMultiplier` and `Playback.maxSpeedMultiplier`.

To control playback position, use `Playback.currentPosition`, which represents the elapsed time in milliseconds from the beginning of the log or simulation. This allows you to skip to any point in the playback.

You also have access to supplementary metadata, such as:

- `Playback.logPath` – the path to the log file being executed

- `Playback.route` – the route being simulated (if applicable)

## Tracking positions

Positions from a `DataSource` can be tracked on a map by rendering a marker polyline between relevant map links points. This is done by using the `MapViewExtensions` class member of `GemMapController`.

The following code illustrates the functionality shown in the screenshot above.
```dart
final mapViewExtensions = controller.extensions;

final err = mapViewExtensions.startTrackPositions(
    updatePositionMs: 500,
    settings: MarkerCollectionRenderSettings(
    polylineInnerColor: Colors.red,
    polylineOuterColor: Colors.yellow,
    polylineInnerSize: 3.0,
    polylineOuterSize: 2.0),
dataSource: dataSource);

// other code ...

mapViewExtensions.stopTrackPositions();
```

| **Method**                    |                 **Parameters**                      | **Return type**            |
|-------------------------------|-----------------------------------------------------|----------------------------|
| **startTrackPositions**       |     - `updatePositionMs`: The tracked position collection update frequency. High frequency may decrease rendering performances on low end devices  - `MarkerCollectionRenderSettings`: The markers collection rendering settings in the map view   - `DataSource?`: The DataSource object which positions are tracked |     `GemError`             |
| **stopTrackPositions**        |                                                   |     - `GemError.success` on success    - `GemError.notFound` if tracking is not started    |
| **isTrackedPositions**        |                                                   |        `bool`              |
| **trackedPositions**       |                                                   |     `List<Coordinates>`    |

If the `dataSource` parameter is left null, tracking will use the current `DataSource` set in `PositionService`. If no `DataSource` is set in `PositionService`, `GemError.notFound` will be returned.

### Getting tracked positions

After calling `MapViewExtensions.startTrackPositions`, you can retrieve the tracked positions later using `trackedPositions` getter. This method returns a list of Coordinates that are used to render the path polyline on `GemMap`.
```dart
final mapViewExtensions = controller.extensions;

mapViewExtensions.trackedPositions;

// other code ...

mapViewExtensions.stopTrackPositions();
```

Calling the `trackedPositions` getter **after** the `stopTrackPositions` is called will result in returning an empty list.

## Relevant examples demonstrating sensors and data source related features

- [Recorder](/examples/routing-navigation/recorder)

- [Record NMEA](/examples/routing-navigation/record-nmea)

- [Recorder in Background](/examples/routing-navigation/background-location)
