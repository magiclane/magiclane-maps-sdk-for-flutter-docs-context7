---
description: Documentation for Areas Alarms
title: Areas Alarms
---

# Areas alarms

This example demonstrates how to build a Flutter app using the Maps SDK which notifies the user when he enters or exits a geographic area. It can be used with any type of area that implements the `GeographicArea` interface.

## How It Works

The example app highlights the following features:

- Display a map with a round polygon marker.

- Calculate route.

- Display route.

- Start simulation.

- Alarm service usage with area monitoring.

### UI and Map Integration

The following code builds the UI with a `GemMap` widget and an app bar that includes buttons for calculating and simulating navigation on route as well as canceling the navigation. Once the position tracker enters or exits the `CircleGeographicArea` a bottom notification panel will be displayed.
```dart
const projectApiToken = String.fromEnvironment('GEM_TOKEN');
void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Areas Alarms',
      home: MyHomePage(),
    );
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key});

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  late GemMapController _mapController;

  bool _areRoutesBuilt = false;
  bool _isSimulationActive = false;

  // We use the progress listener to cancel the route calculation.
  TaskHandler? _routingHandler;

  TaskHandler? _navigationHandler;
  AlarmService? _alarmService;
  AlarmListener? _alarmListener;

  String? _areaNotification;

  @override
  void dispose() {
    GemKit.release();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text(
          "Areas Alarms",
          style: TextStyle(color: Colors.white),
        ),
        backgroundColor: Colors.deepPurple[900],
        actions: [
          if (!_isSimulationActive && _areRoutesBuilt)
            IconButton(
              onPressed: _startSimulation,
              icon: const Icon(Icons.play_arrow, color: Colors.white),
            ),
          if (_isSimulationActive)
            IconButton(
              onPressed: _stopSimulation,
              icon: const Icon(Icons.stop, color: Colors.white),
            ),
          if (!_areRoutesBuilt)
            IconButton(
              onPressed: () => _onBuildRouteButtonPressed(context),
              icon: const Icon(Icons.route, color: Colors.white),
            ),
        ],
      ),
      body: Stack(
        children: [
          GemMap(
            key: ValueKey("GemMap"),
            onMapCreated: _onMapCreated,
            appAuthorization: projectApiToken,
          ),
          if (_areaNotification != null)
            Positioned(
              bottom: MediaQuery.of(context).padding.bottom + 10,
              left: 0,
              child: BottomAlarmPanel(alarmNotification: _areaNotification),
            ),
        ],
      ),
      resizeToAvoidBottomInset: false,
    );
  }

  void _onMapCreated(GemMapController controller) {
    _mapController = controller;

    // Draw area on map
    final marker = Marker();
    final circleAreaCoords = generateCircleCoordinates(Coordinates(latitude: 50.92396, longitude: 9.54976), 200);

    for (final coord in circleAreaCoords) {
      marker.add(coord);
    }

    final markerCollection = MarkerCollection(markerType: MarkerType.polygon, name: "Circle");
    markerCollection.add(marker);

    _mapController.preferences.markers.add(markerCollection,
        settings: MarkerCollectionRenderSettings(polygonFillColor: const Color.fromARGB(111, 210, 104, 102)));
  }

  // Custom method for calling calculate route and displaying the results.
  void _onBuildRouteButtonPressed(BuildContext context) {
    // Define the departure.
    final departureLandmark = Landmark.withLatLng(
      latitude: 50.92899490001731,
      longitude: 9.544136681645025,
    );

    // Define the destination.
    final destinationLandmark = Landmark.withLatLng(
      latitude: 50.919902402432946,
      longitude: 9.55855522546262,
    );
    // Define the route preferences.
    final routePreferences = RoutePreferences();
    _showSnackBar(context, message: 'The route is calculating.');

    // Calling the calculateRoute SDK method.
    // (err, results) - is a callback function that gets called when the route computing is finished.
    // err is an error enum, results is a list of routes.
    _routingHandler = RoutingService.calculateRoute(
      [departureLandmark, destinationLandmark],
      routePreferences,
      (err, routes) async {
        // If the route calculation is finished, we don't have a progress listener anymore.
        _routingHandler = null;

        ScaffoldMessenger.of(context).clearSnackBars();

        // If there aren't any errors, we display the routes.
        if (err == GemError.success) {
          // Get the routes collection from map preferences.
          final routesMap = _mapController.preferences.routes;

          // Display the routes on map.
          for (final route in routes) {
            routesMap.add(route, route == routes.first);
          }

          // Center the camera on routes.
          _mapController.centerOnRoutes(routes: routes);
        }
        setState(() {
          _areRoutesBuilt = true;
        });
      },
    );
  }

  // Method for starting the simulation and following the position,
  void _startSimulation() {
    final routes = _mapController.preferences.routes;

    _mapController.preferences.routes.clearAllButMainRoute();

    if (routes.mainRoute == null) {
      _showSnackBar(context, message: "No main route available");
      return;
    }

    // Registering callback for area corssing
    _alarmListener = AlarmListener(
      onBoundaryCrossed: (enteredAreas, exitedAreas) {
        if (enteredAreas.isNotEmpty) {
          setState(() {
            _areaNotification = "Entered area: ${enteredAreas.first}";
          });
        } else {
          setState(() {
            _areaNotification = "Exited area: ${exitedAreas.first}";
          });
        }
      },
    );

    // Set the alarms service with the listener
    _alarmService = AlarmService(_alarmListener!);

    _alarmService!.monitorArea(
        CircleGeographicArea(radius: 200, centerCoordinates: Coordinates(latitude: 50.92396, longitude: 9.54976)),
        id: "Test area");

    _navigationHandler = NavigationService.startSimulation(
      routes.mainRoute!,
      null,
      onNavigationInstruction: (instruction, events) {
        setState(() {
          _isSimulationActive = true;
        });
      },
      onDestinationReached: (landmark) {
        _stopSimulation();
        _cancelRoute();
      },
      onError: (error) {
        // If the navigation has ended or if and error occurred while navigating, remove routes and reset alarm notification.
        setState(() {
          _isSimulationActive = false;
          _areaNotification = null;
          _cancelRoute();
        });

        if (error != GemError.cancel) {
          _stopSimulation();
        }
        return;
      },
    );

    // Set the camera to follow position.
    _mapController.startFollowingPosition();
  }

  // Method for removing the routes from display,
  void _cancelRoute() {
    // Remove the routes from map.
    _mapController.preferences.routes.clear();

    if (_routingHandler != null) {
      // Cancel the navigation.
      RoutingService.cancelRoute(_routingHandler!);
      _routingHandler = null;
    }

    setState(() {
      _areRoutesBuilt = false;
    });
  }

  // Method to stop the simulation and remove the displayed routes,
  void _stopSimulation() {
    // Cancel the navigation.
    NavigationService.cancelNavigation(_navigationHandler!);
    _navigationHandler = null;
    _areaNotification = null;

    _cancelRoute();

    setState(() => _isSimulationActive = false);
  }
}
```

### Bottom Alarm Panel
```dart
class BottomAlarmPanel extends StatelessWidget {
  final String? alarmNotification;

  const BottomAlarmPanel({
    super.key,
    required this.alarmNotification,
  });

  @override
  @override
  Widget build(BuildContext context) {
    return Container(
      decoration: BoxDecoration(
        color: Colors.white,
        borderRadius: const BorderRadius.all(Radius.circular(20)),
        boxShadow: [
          BoxShadow(
            color: Colors.grey.withValues(alpha: 0.5),
            spreadRadius: 5,
            blurRadius: 7,
            offset: const Offset(0, 3),
          ),
        ],
      ),
      width: MediaQuery.of(context).size.width - 20,
      height: 50,
      margin: const EdgeInsets.symmetric(horizontal: 10),
      padding: const EdgeInsets.symmetric(horizontal: 15),
      child: Center(
          child: Text(
        alarmNotification!,
        style: const TextStyle(
          color: Colors.black,
          fontSize: 24,
          fontWeight: FontWeight.w500,
        ),
      )),
    );
  }
}
```

### Utility Functions
```dart
// Method to show message in case calculate route is not finished,
void _showSnackBar(
  BuildContext context, {
  required String message,
  Duration duration = const Duration(hours: 1),
}) {
  final snackBar = SnackBar(content: Text(message), duration: duration);
  ScaffoldMessenger.of(context).showSnackBar(snackBar);
}

// Method for generating coordinates in a circle shape
List<Coordinates> generateCircleCoordinates(
  Coordinates center,
  double radiusMeters, {
  int numberOfPoints = 36,
}) {
  const earthRadius = 6371000; // in meters
  final centerLatRad = center.latitude * pi / 180;
  final centerLonRad = center.longitude * pi / 180;
  final coordinates = <Coordinates>[];

  for (var i = 0; i < numberOfPoints; i++) {
    final angle = 2 * pi * i / numberOfPoints;
    final deltaLat = (radiusMeters / earthRadius) * cos(angle);
    final deltaLon = (radiusMeters / (earthRadius * cos(centerLatRad))) * sin(angle);
    final pointLat = (centerLatRad + deltaLat) * 180 / pi;
    final pointLon = (centerLonRad + deltaLon) * 180 / pi;
    coordinates.add(Coordinates(latitude: pointLat, longitude: pointLon));
  }

  return coordinates;
}
```


