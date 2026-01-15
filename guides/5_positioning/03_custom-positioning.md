---
description: Documentation for Custom Positioning
title: Custom Positioning
---

# Custom positioning

Set a custom data source with the PositionService to dynamically manage and simulate location data. Use external or simulated positioning data instead of traditional GPS signals—ideal for testing or custom tracking solutions.

Using a custom data source eliminates the need for location permission management.

---

## Create custom data source

Integrate a custom data source with the PositionService to manage and simulate location data. The custom data source allows external or simulated position data instead of real GPS signals:
```dart
// Create a custom data source.
final dataSource = DataSource.createExternalDataSource([DataType.position]);

if (dataSource == null){
  showSnackbar("The datasource could not be created");
  return;
}

// Positions will be provided from the data source.
PositionService.setExternalDataSource(dataSource);

// Start the data source.
dataSource.start();

// Push the first position in the data source
dataSource.pushData(
  SenseDataFactory.producePosition(
    acquisitionTime: DateTime.now(),
    latitude: 48.85682,
    longitude: 2.34375,
    altitude: 0,
    course: 0,
    speed: 0,
  ),
);

// Optional, usually done if we want the map to take into
// account the current position.
controller.startFollowingPosition();

while (true) {
  await Future<void>.delayed(Duration(milliseconds: 50));

  // Provide latitude, longitude, heading, speed.
  double lat = 45;
  double lon = 10;
  double head = 0;
  double speed = 0;

  // Add each position to data source.
  dataSource.pushData(
    SenseDataFactory.producePosition(
      acquisitionTime: DateTime.now(),
      latitude: lat,
      longitude: lon,
      altitude: 0,
      course: head,
      speed: speed,
    )
  );
}
```

**How it works:**

- **Create and register a custom data source** - Create a custom DataSource object configured to handle position data. Register it with the PositionService, overriding the default GPS-based data provider

- **Start the data source** - Activate the custom data source by calling the `start()` method. Once started, it's ready to accept and process location data

- **Push initial position data** - Send an initial position to the data source using the `pushData` method. This includes latitude, longitude, altitude, heading, speed, and timestamp

- **Enable map follow mode** - The `startFollowingPosition` method ensures the map camera follows the position tracker. The map view adjusts automatically as new position updates arrive

- **Update location data in real-time** - A loop continuously generates and pushes simulated position updates at regular intervals (every 50 milliseconds). This allows the application to simulate movement or integrate location data from custom sources

---

## Improve custom data source positions

While latitude, longitude, and timestamp may suffice for some use cases, this data may not offer sufficient accuracy during navigation. The system might occasionally register incorrect turns or unexpected deviations. To improve precision, use the `heading` field of `ExternalPositionData` to indicate the direction of movement, which is factored into positioning calculations.

### Calculate heading

Calculate the heading using the current coordinate and the next coordinate:
```dart
double _getHeading(Coordinates from, Coordinates to) {
  final dx = to.longitude - from.longitude;
  final dy = to.latitude - from.latitude;

  const radianToDegree = 57.2957795;

  final val = atan2(dx, dy) * radianToDegree;
  if (val < 0) val + 360;

  return val;
}
```

### Calculate speed

Compute the `speed` field from the `ExternalPositionData` by dividing the distance between two coordinates by the duration of movement between them. Calculate the distance using the `distance` method from the `Coordinate` class: 
```dart
double _getSpeed(Coordinates from, Coordinates to, DateTime timestampAtFrom, DateTime timestampAtTo) {
  final timeDiff = timestampAtTo.difference(timestampAtFrom).inSeconds;
  final distance = from.distance(to);

  if (timeDiff == 0) {
    return 0;
  }

  return distance / timeDiff;
}
```

If the coordinates to be pushed in the custom data source are not known, extrapolate them based on previous values.

---

## Remove the custom data source

Remove the data source when no longer needed:
```dart
DataSource dataSource = DataSource.createExternalDataSource([DataType.position]);

if (dataSource == null){
  showSnackbar("The datasource could not be created");
  return;
}

PositionService.setExternalDataSource(dataSource);
dataSource.start();

// Do something with the data source...

// Stop the data source.
dataSource.stop();

// Remove the data source from the position service.
PositionService.removeDataSource();
```

Stop the data source and remove it from the position service when finished. Otherwise, unexpected problems can occur when trying to use other data sources (live or custom).

---

## Create simulation data source

Integrate a simulation data source with the PositionService to simulate location data by following a given route. This data source behaves as a route simulation:
```dart
// Create a simulation data source.
final dataSource = DataSource.createSimulationDataSource(route);

// Positions will be provided from the data source.
PositionService.setExternalDataSource(dataSource);

// Start the data source.
dataSource.start();

```

**How it works:**

- **Create and register a simulation data source** - Create a DataSource object configured to handle position data. Register it with the PositionService, overriding the default GPS-based data provider

- **Start the data source** - Activate the data source by calling the `start()` method. Once started, it begins simulating the given route

---

## Remove the simulation data source

Remove the data source when no longer needed:
```dart
DataSource dataSource = DataSource.createSimulationDataSource(route);
PositionService.setExternalDataSource(dataSource);
dataSource.start();

// Do something with the data source...

// Stop the data source.
dataSource.stop();

// Remove the data source from the position service.
PositionService.removeDataSource();
```

Stop the data source and remove it from the position service when finished. Otherwise, unexpected problems can occur when trying to use other data sources (live or custom).

---

## Create log data source

Integrate a log data source with the PositionService to replay location data from a predefined log file. This enables the application to replay location data, ensuring uniformity across different runs:
```dart
// Create a simulation data source.
final dataSource = DataSource.createLogDataSource(logFile);

// Positions will be provided from the data source.
PositionService.setExternalDataSource(dataSource);

```

The log data source starts automatically when created. No need to call the `start()` method.

**How it works:**

- **Create and register a log data source** - Create a DataSource object configured to handle position data. Register it with the PositionService, overriding the default GPS-based data provider

- **Start the data source** - The data source activates automatically and begins replaying the recorded data

---

## Remove the log data source

Remove the data source when no longer needed:
```dart
DataSource dataSource = DataSource.createLogDataSource(logFile);
PositionService.setExternalDataSource(dataSource);

// Do something with the data source...

// Stop the data source.
dataSource.stop();

// Remove the data source from the position service.
PositionService.removeDataSource();
```

The log data source cannot be of type `.gm`. This file type is not supported in the public SDK. Use another format, such as `gpx`, `nmea`, or `kml`, that can be exported from the `.gm` file.

Stop the data source and remove it from the position service when finished. Otherwise, unexpected problems can occur when trying to use other data sources (live or custom).

---

## Relevant example demonstrating custom positioning related features

- [External Position Source Navigation](/examples/routing-navigation/external-position-source-navigation)

- [Data Source Listeners](/examples/routing-navigation/datasource_listeners)
