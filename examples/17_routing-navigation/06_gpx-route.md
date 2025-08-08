---
description: Documentation for Gpx Route
title: Gpx Route
---

# GPX Route

This example demonstrates how to calculate a route based on GPX data, render and center the route on an interactive map, and navigate along the route.

## How It Works

The example app demonstrates the following features:

- Import GPX data to calculate and display routes.

- Render the calculated route on the map and center the camera to fit the entire route.

- Simulate navigation along the route.

- Display notifications and format route labels.

### Saving Assets

Before running the app, ensure that you save the necessary files (gpx files) into the assets directory.

Update your pubspec.yaml file to include these assets:
```yaml
flutter:
  assets:
    - assets/
```

### Copying the GPX File

This function copies the recorded_route.gpx file from the assets directory to the app’s documents directory:
```dart
Future<void> _copyGpxToAppDocsDir() async {
  if (!kIsWeb) {
    final docDirectory = await getApplicationDocumentsDirectory();
    final gpxFile = File('${docDirectory.path}/recorded_route.gpx');
    final fileBytes = await rootBundle.load('assets/recorded_route.gpx');
    final buffer = fileBytes.buffer;
    await gpxFile.writeAsBytes(
      buffer.asUint8List(fileBytes.offsetInBytes, fileBytes.lengthInBytes),
    );
  }
}
```

The function ensures that the GPX file is available in the app’s documents directory, ready for use during runtime.

### Importing GPX Data and Calculating Routes

This function reads GPX data from the file, calculates the routes, and displays them on the map:
```dart
Future<void> _importGPX() async {
  _showSnackBar(context, message: 'The route is calculating.');

  List<Landmark> landmarkList = [];

  if (kIsWeb) {
    final imageBytes = await rootBundle.load('assets/recorded_route.gpx');
    final buffer = imageBytes.buffer;
    final pathData = buffer.asUint8List(
      imageBytes.offsetInBytes,
      imageBytes.lengthInBytes,
    );

    final gemPath = Path.create(data: pathData, format: PathFileFormat.gpx);
    landmarkList = gemPath.toLandmarkList();
  } else {
    final docDirectory = await getApplicationDocumentsDirectory();
    final gpxFile = File('${docDirectory.path}/recorded_route.gpx');

    if (!await gpxFile.exists()) {
      print('GPX file does not exist (${gpxFile.path})');
      return;
    }

    final bytes = await gpxFile.readAsBytes();
    final pathData = Uint8List.fromList(bytes);

    final gemPath = Path.create(data: pathData, format: PathFileFormat.gpx);
    landmarkList = gemPath.toLandmarkList();
  }

  print("GPX Landmarklist size: ${landmarkList.length}");

  final routePreferences = RoutePreferences(
    transportMode: RouteTransportMode.bicycle,
  );

  RoutingService.calculateRoute(landmarkList, routePreferences, (
    err,
    routes,
  ) {
    ScaffoldMessenger.of(context).clearSnackBars();

    if (err == GemError.success) {
      final routesMap = _mapController.preferences.routes;

      for (final route in routes) {
        routesMap.add(
          route,
          route == routes.first,
          label: route.getMapLabel(),
        );
      }

      _mapController.centerOnRoutes(routes: routes);

      setState(() {
        _areRoutesBuilt = true;
      });
    }
  });
  _isGpxDataLoaded = true;
}
```

When the “Calculate Route” button is pressed, this function reads the GPX file from the app’s documents directory and calculates a route based on the landmarks (waypoints) extracted from the GPX data.
The routes are displayed on the map and the camera is centered to ensure all routes fit within the viewport.

### Starting and Stopping the Simulation

The simulation can be started or stopped using the following functions:
```dart
void _startSimulation() {
  if (_isSimulationActive) return;
  if (!_isGpxDataLoaded) return;

  final routes = _mapController.preferences.routes;

  if (routes.mainRoute == null) {
    _showSnackBar(context, message: "No route available");
    return;
  }

  _navigationHandler = NavigationService.startSimulation(routes.mainRoute!, (
    eventType,
    instruction,
  ) {
  }, speedMultiplier: 2);

  _mapController.startFollowingPosition();

  setState(() => _isSimulationActive = true);
}

void _stopSimulation() {
  _mapController.preferences.routes.clear();

  setState(() => _areRoutesBuilt = false);

  if (_isSimulationActive) {
    NavigationService.cancelNavigation(_navigationHandler!);
    _navigationHandler = null;

    setState(() => _isSimulationActive = false);
  }
}
```

The _startSimulation function initiates the navigation simulation along the main route, with the camera following the simulated position. The _stopSimulation function stops the simulation and clears the routes from the map.

### Utility Functions

Utility functions are also included to display messages and format route labels:
```dart
void _showSnackBar(BuildContext context,
    {required String message, Duration duration = const Duration(hours: 1)}) {
  final snackBar = SnackBar(
    content: Text(message),
    duration: duration,
  );

  ScaffoldMessenger.of(context).showSnackBar(snackBar);
}
```

The _showSnackBar method displays messages during route calculation. The RouteExtension is an extension that formats route labels to display the total distance and duration.

### Displaying Route Information

This code defines an extension on the Route class that calculates and formats the distance and duration of the route for display on the map.
```dart
extension RouteExtension on Route {
  String getMapLabel() {
    final totalDistance = getTimeDistance().unrestrictedDistanceM +
        getTimeDistance().restrictedDistanceM;
    final totalDuration =
        getTimeDistance().unrestrictedTimeS + getTimeDistance().restrictedTimeS;

    return '${_convertDistance(totalDistance)} \n${_convertDuration(totalDuration)}';
  }

  String _convertDistance(int meters) {
    if (meters >= 1000) {
      double kilometers = meters / 1000;
      return '${kilometers.toStringAsFixed(1)} km';
    } else {
      return '${meters.toString()} m';
    }
  }

  String _convertDuration(int seconds) {
    int hours = seconds ~/ 3600;
    int minutes = (seconds % 3600) ~/ 60;

    String hoursText = (hours > 0) ? '$hours h ' : '';
    String minutesText = '$minutes min';

    return hoursText + minutesText;
  }
}
```


