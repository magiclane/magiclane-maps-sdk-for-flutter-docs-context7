---
description: Documentation for Custom Positioning
title: Custom Positioning
---

# Custom positioning

The Maps SDK for Flutter allows setting custom data source with the PositionService to dynamically manage and simulate location data. This approach allows for external or simulated positioning data, providing flexibility beyond traditional GPS signals, and is ideal for testing or custom tracking solutions.

Utilizing a custom data source eliminates the need for the previously discussed location permission management.

## Create custom data source

The following code snippet illustrates how to integrate a custom data source with the PositionService to manage and simulate location data dynamically. Instead of relying on real GPS signals, the custom data source provides flexibility by allowing external or simulated position data to be used in the application.
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

How It Works:

- **Creating and Registering a Custom Data Source**: A custom DataSource object is created and configured to handle position data. This data source is registered with the PositionService, overriding the default GPS-based data provider. This allows the application to retrieve location updates from the custom source.

- **Starting the Data Source**: The custom data source is activated by calling the start() method. Once started, it becomes ready to accept and process location data that is pushed into it.

- **Pushing Initial Position Data**: An initial position is sent to the data source using the pushData method. This data includes details such as latitude, longitude, altitude, heading, speed, and a timestamp. It acts as a starting point for tracking the location.

- **Enabling Map Follow Mode**: The startFollowingPosition method ensures the map camera follows the position tracker. As the custom data source provides new position updates, the map view adjusts automatically to keep the position tracker in focus.

- **Updating Location Data in Real-Time**: A loop continuously generates and pushes simulated position updates to the data source at regular intervals (every 50 milliseconds). These updates include coordinates, heading, and speed. This dynamic update mechanism allows the application to simulate movement or integrate location data from custom sources, such as a mock GPS or external tracking systems.

## Improve custom data source positions

While providing latitude, longitude, and timestamp may suffice for some use cases, this data may not offer sufficient accuracy, particularly during navigation. In such cases, the system might occasionally register incorrect turns or unexpected deviations. To improve precision, the ``heading`` field of ``ExternalPositionData`` is utilized to indicate the direction of movement, which is factored into the positioning calculations.

### Calculate heading

A simple function to calculate the heading knowing the current coordinate and the next coordinate is presented below:
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

The ``speed`` field from the ``ExternalPositionData`` can be computed using by dividing the distance between the two coordinates by the duration of the movement between the two coordinates. The distance can be computed using the ``distance`` method from the ``Coordinate`` class. 
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

If the coordinates to be pushed in the custom data source are not known they can be extrapolated based on the previous values.

## Remove the custom datasource

To remove the data source once we don't need it anymore, we can proceed in the following way:
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

It's important to stop the data source and remove it from the position service once work is finished with it.
Otherwise there can be unexpected problems especially when trying to use other data sources (live or custom). 

## Create simulation data source

The following code snippet illustrates how to integrate a simulation data source with the PositionService to manage and simulate location data dynamically. Instead of relying on real GPS signals, the simulation data source provides flexibility by following a given route. This kind of data source behaves as a route simulation.
```dart
// Create a simulation data source.
final dataSource = DataSource.createSimulationDataSource(route);

// Positions will be provided from the data source.
PositionService.setExternalDataSource(dataSource);

// Start the data source.
dataSource.start();

```

How It Works:

- **Creating and Registering a Custom Data Source**: A custom DataSource object is created and configured to handle position data. This data source is registered with the PositionService, overriding the default GPS-based data provider. This allows the application to retrieve location updates from the custom source.

- **Starting the Data Source**: The custom data source is activated by calling the start() method. Once started, it starts simulating the given route.

## Remove the simulation datasource

To remove the data source once we don't need it anymore, we can proceed in the following way:
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

It's important to stop the data source and remove it from the position service once work is finished with it.
Otherwise there can be unexpected problems especially when trying to use other data sources (live or custom). 

## Create log data source

The following code snippet illustrates how to integrate a log data source with the PositionService to manage and simulate location data dynamically. Instead of relying on real GPS signals, the log data source provides flexibility by mirroring a given log file. This enables the application to replay location data from a stable and predefined log, ensuring uniformity across different runs.
```dart
// Create a simulation data source.
final dataSource = DataSource.createLogDataSource(logFile);

// Positions will be provided from the data source.
PositionService.setExternalDataSource(dataSource);

```

The log data source will start automatically when created, so there is no need to call the start() method.

How It Works:

- **Creating and Registering a Custom Data Source**: A custom DataSource object is created and configured to handle position data. This data source is registered with the PositionService, overriding the default GPS-based data provider. This allows the application to retrieve location updates from the custom source.

- **Starting the Data Source**: The custom data source is activated by calling the start() method. Once started, it starts simulating the recorded data.

## Remove the log datasource

To remove the data source once we don't need it anymore, we can proceed in the following way:
```dart
DataSource dataSource = DataSource.createLogDataSource(logFile);
PositionService.setExternalDataSource(dataSource);

// Do something with the data source...

// Stop the data source.
dataSource.stop();

// Remove the data source from the position service.
PositionService.removeDataSource();
```

The log data source cannot be of type `.gm`. Using this file type is not supported in the public SDK. You can use another format, such as `gpx`, `nmea` or `kml`, that can be exported from the `.gm` file.

It's important to stop the data source and remove it from the position service once work is finished with it.
Otherwise there can be unexpected problems especially when trying to use other data sources (live or custom). 

## Relevant example demonstrating custom positioning related features

- [External Position Source Navigation](/examples/routing-navigation/external-position-source-navigation)

- [Data Source Listeners](/examples/routing-navigation/datasource_listeners)
