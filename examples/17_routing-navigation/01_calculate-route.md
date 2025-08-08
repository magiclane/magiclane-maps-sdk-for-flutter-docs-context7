---
description: Documentation for Calculate Route
title: Calculate Route
---

# Calculate Route

This example demonstrates how to create a Flutter app that calculates a route between two locations and displays it on a map using Maps SDK for Flutter.

## How It Works

The example app demonstrates the following features:

- Calculate a route between two landmarks.

- Display the route on a map and allow user interaction with the route.

- Provide options to cancel route calculation or clear the routes from the map.

### UI and Map Integration

This code sets up the basic structure of the app, including the map and the app bar. It also provides buttons in the app bar for building, canceling, and clearing routes.
```dart
class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key});

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  late GemMapController _mapController;

  TaskHandler? _routingHandler;
  List<Route>? _routes;

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
        title: const Text('Calculate Route',
            style: TextStyle(color: Colors.white)),
        actions: [
          // Routes are not built.
          if (_routingHandler == null && _routes == null)
            IconButton(
              onPressed: () => _onBuildRouteButtonPressed(context),
              icon: const Icon(
                Icons.route,
                color: Colors.white,
              ),
            ),
          // Routes calculating is in progress.
          if (_routingHandler != null)
            IconButton(
              onPressed: () => _onCancelRouteButtonPressed(),
              icon: const Icon(
                Icons.stop,
                color: Colors.white,
              ),
            ),
          // Routes calculating is finished.
          if (_routes != null)
            IconButton(
              onPressed: () => _onClearRoutesButtonPressed(),
              icon: const Icon(
                Icons.clear,
                color: Colors.white,
              ),
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

### Route Calculation and Map Interaction 

This code handles the calculation of routes between two landmarks, displays the routes on the map, and provides options to cancel or clear the routes. The map is centered on the calculated routes, and a label showing the distance and duration is displayed.
```dart
// The callback for when map is ready to use.
void _onMapCreated(GemMapController controller) {
  // Save controller for further usage.
  _mapController = controller;

  // Register route tap gesture callback.
  _registerRouteTapCallback();
}

void _onBuildRouteButtonPressed(BuildContext context) {
  // Define the departure.
  final departureLandmark =
      Landmark.withLatLng(latitude: 48.85682, longitude: 2.34375);

  // Define the destination.
  final destinationLandmark =
      Landmark.withLatLng(latitude: 50.84644, longitude: 4.34587);

  // Define the route preferences.
  final routePreferences = RoutePreferences();

  _showSnackBar(context, message: "The route is being calculated.");

  // Calling the calculateRoute SDK method.
  // (err, results) - is a callback function that gets called when the route computing is finished.
  // err is an error enum, results is a list of routes.

  _routingHandler = RoutingService.calculateRoute(
      [departureLandmark, destinationLandmark], routePreferences,
      (err, routes) {
    // If the route calculation is finished, we don't have a progress listener anymore.
    _routingHandler = null;
    ScaffoldMessenger.of(context).clearSnackBars();

    // If there aren't any errors, we display the routes.
    if (err == GemError.success) {
      // Get the routes collection from map preferences.
      final routesMap = _mapController.preferences.routes;

      // Display the routes on map.
      for (final route in routes!) {
        routesMap.add(route, route == routes.first,
            label: route.getMapLabel());
      }

      // Center the camera on routes.
      _mapController.centerOnRoutes(routes: routes);
      setState(() {
        _routes = routes;
      });
    }
  });

  setState(() {});
}

void _onClearRoutesButtonPressed() {
  // Remove the routes from map.
  _mapController.preferences.routes.clear();

  setState(() {
    _routes = null;
  });
}

void _onCancelRouteButtonPressed() {
  // If we have a progress listener we cancel the route calculation.
  if (_routingHandler != null) {
    RoutingService.cancelRoute(_routingHandler!);

    setState(() {
      _routingHandler = null;
    });
  }
}
```

### Route Selection

This code enables the user to select a specific route on the map by tapping on it. The selected route becomes the main route displayed.
```dart
// In order to be able to select an alternative route, we have to register the route tap gesture callback.
void _registerRouteTapCallback() {
  // Register the generic map touch gesture.
  _mapController.registerTouchCallback((pos) async {
    // Select the map objects at given position.
    _mapController.setCursorScreenPosition(pos);

    // Get the selected routes.
    final routes = _mapController.cursorSelectionRoutes();

    // If there is a route at position, we select it as the main one on the map.
    if (routes.isNotEmpty) {
      _mapController.preferences.routes.mainRoute = routes[0];
    }
  });
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


