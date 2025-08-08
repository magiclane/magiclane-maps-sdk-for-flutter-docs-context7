---
description: Documentation for Finger Route
title: Finger Route
---

# Finger Route

This example demonstrates how to create a Flutter app that allows users to draw a route on a map using their finger, calculates the route based on the drawn waypoints, and displays it using the Maps SDK for Flutter.

## How It Works

The example app demonstrates the following features:

- Allow users to draw a route on the map with their finger.

- Calculate a route based on the drawn waypoints.

- Display the route on a map and provide options to cancel or clear the routes.

### UI and Map Integration

This code sets up the basic structure of the app, including the map and the app bar. It also provides buttons in the app bar for drawing, building, canceling, and clearing routes.
```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Finger Route',
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

  TaskHandler? _routingHandler;
  bool _areRoutesBuilt = false;
  bool _isInDrawingMode = false;

  @override
  void dispose() {
    GemKit.release();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        backgroundColor: Colors.deepPurple[900],
        title: const Text('Finger Route', style: TextStyle(color: Colors.white)),
        actions: [
          // Enable drawing mode.
          if (_routingHandler == null && !_areRoutesBuilt && !_isInDrawingMode)
            IconButton(
              onPressed: () => _onDrawPressed(),
              icon: const Icon(CupertinoIcons.hand_draw, color: Colors.white),
            ),
          // Build route from drawn waypoints.
          if (_routingHandler == null && !_areRoutesBuilt && _isInDrawingMode)
            IconButton(
              onPressed: () => _onBuildRouteButtonPressed(context),
              icon: const Icon(Icons.done, color: Colors.white),
            ),
          // Cancel route calculation.
          if (_routingHandler != null)
            IconButton(
              onPressed: () => _onCancelRouteButtonPressed(),
              icon: const Icon(Icons.stop, color: Colors.white),
            ),
          // Clear the drawn routes.
          if (_areRoutesBuilt)
            IconButton(
              onPressed: () => _onClearRoutesButtonPressed(),
              icon: const Icon(Icons.clear, color: Colors.white),
            ),
        ],
      ),
      body: GemMap(
        key: ValueKey("GemMap"),
        onMapCreated: _onMapCreated,
        appAuthorization: projectApiToken,
      ),
    );
  }
```

### Drawing and Route Calculation

This code handles drawing waypoints on the map, calculating the route based on those waypoints, and provides options to cancel or clear the routes. The map is centered on the calculated routes, and a label showing the distance and duration is displayed.
```dart
// The callback for when the map is ready to use.
void _onMapCreated(GemMapController controller) {
  // Save controller for further usage.
  _mapController = controller;
}

void _onDrawPressed() {
  _mapController.enableDrawMarkersMode();
  setState(() {
    _isInDrawingMode = true;
  });
}

void _onBuildRouteButtonPressed(BuildContext context) {
  final waypoints = _mapController.disableDrawMarkersMode();

  // Define the route preferences.
  final routePreferences = RoutePreferences(
      accurateTrackMatch: false, ignoreRestrictionsOverTrack: true);

  _showSnackBar(context, message: "The route is being calculated.");

  _routingHandler = RoutingService.calculateRoute(waypoints, routePreferences,
      (err, routes) {
    setState(() {
      _routingHandler = null;
      _isInDrawingMode = false;
    });

    ScaffoldMessenger.of(context).clearSnackBars();

    if (err == GemError.success) {
      final routesMap = _mapController.preferences.routes;

      for (final route in routes!) {
        routesMap.add(route, route == routes.first,
            label: route.getMapLabel());
      }

      _mapController.centerOnRoutes(routes: routes);
      setState(() {
        _areRoutesBuilt = true;
      });
    }
  });

  setState(() {});
}

void _onClearRoutesButtonPressed() {
  // Remove the routes from map.
  _mapController.preferences.routes.clear();

  setState(() {
    _areRoutesBuilt = false;
  });
}

void _onCancelRouteButtonPressed() {
  // If we have a progress listener, cancel the route calculation.
  if (_routingHandler != null) {
    RoutingService.cancelRoute(_routingHandler!);

    setState(() {
      _routingHandler = null;
    });
  }
}
```

### Displaying Route Information

This code defines an extension on the Route class that calculates and formats the distance and duration of the route for display on the map.
```dart
// Define an extension for route for calculating the route label which will be displayed on map.
extension RouteExtension on Route {
  String getMapLabel() {
    final totalDistance = getTimeDistance().unrestrictedDistanceM +
        getTimeDistance().restrictedDistanceM;
    final totalDuration =
        getTimeDistance().unrestrictedTimeS + getTimeDistance().restrictedTimeS;

    return '${_convertDistance(totalDistance)} \n${_convertDuration(totalDuration)}';
  }

  // Utility function to convert the meters distance into a suitable format.
  String _convertDistance(int meters) {
    if (meters >= 1000) {
      double kilometers = meters / 1000;
      return '${kilometers.toStringAsFixed(1)} km';
    } else {
      return '${meters.toString()} m';
    }
  }

  // Utility function to convert the seconds duration into a suitable format.
  String _convertDuration(int seconds) {
    int hours = seconds ~/ 3600; // Number of whole hours
    int minutes = (seconds % 3600) ~/ 60; // Number of whole minutes

    String hoursText = (hours > 0) ? '$hours h ' : ''; // Hours text
    String minutesText = '$minutes min'; // Minutes text

    return hoursText + minutesText;
  }
}
```


