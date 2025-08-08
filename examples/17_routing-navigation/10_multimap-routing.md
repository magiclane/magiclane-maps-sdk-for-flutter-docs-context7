---
description: Documentation for Multimap Routing
title: Multimap Routing
---

# Multi Map Routing

In this guide, you will learn how to implement multi-map routing functionality using the Maps SDK for Flutter. This example demonstrates how to manage routes on two maps simultaneously.

## How it Works

This example demonstrates the following features:

- Interact with and manage routing functionalities on two separate maps within the same application.

- Create and calculate routes with specified waypoints and preferences for each map independently.

### Build the Main Application 

Define the main application widget, MyApp.
```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
        title: 'Multi Map Routing',
        debugShowCheckedModeBanner: false,
        home: MyHomePage());
  }
}
```

### Handle Maps and Routes in the Stateful Widget

Create the stateful widget, MyHomePage , which will handle two maps and their respective routes.
```dart
class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key});

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}
```

### Define State Variables and Methods

Within _MyHomePageState , define the necessary state variables and methods to interact with the maps and manage routes.
```dart
class _MyHomePageState extends State<MyHomePage> {
  late Controller _mapController1;
  late Controller _mapController2;

  TaskHandler? _routingHandler1;
  TaskHandler? _routingHandler2;

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
        title: const Text(
          'Multi Map Routing',
          style: TextStyle(color: Colors.white),
        ),
        leading: IconButton(
          onPressed: _removeRoutes,
          icon: const Icon(Icons.close, color: Colors.white),
        ),
        actions: [
          IconButton(
            onPressed: () => _onBuildRouteButtonPressed(true),
            icon: const Icon(Icons.route, color: Colors.white),
          ),
          IconButton(
            onPressed: () => _onBuildRouteButtonPressed(false),
            icon: const Icon(Icons.route, color: Colors.white),
          ),
        ],
      ),
      body: Column(
        children: [
          Expanded(
            child: Padding(
              padding: const EdgeInsets.all(8.0),
              child: GemMap(key: ValueKey("GemMap"), onMapCreated: _onMap1Created, appAuthorization: projectApiToken),
            ),
          ),
          Expanded(
            child: Padding(
              padding: const EdgeInsets.all(8.0),
              child: GemMap(onMapCreated: _onMap2Created),
            ),
          ),
        ],
      ),
    );
  }

  void _showSnackBar(BuildContext context,
      {required String message, Duration duration = const Duration(hours: 1)}) {
    final snackBar = SnackBar(
      content: Text(message),
      duration: duration,
    );

    ScaffoldMessenger.of(context).showSnackBar(snackBar);
  }

  void _onMap1Created(GemMapController controller) {
    _mapController1 = controller;
  }

  void _onMap2Created(GemMapController controller) {
    _mapController2 = controller;
  }

  void _onBuildRouteButtonPressed(bool isFirstMap) {
    final waypoints = <Landmark>[];
    if (isFirstMap) {
      final departure =
          Landmark.withLatLng(latitude: 37.77903, longitude: -122.41991);
      final destination =
          Landmark.withLatLng(latitude: 37.33619, longitude: -121.89058);

      waypoints.add(departure);
      waypoints.add(destination);
    } else {
      final departure =
          Landmark.withLatLng(latitude: 51.50732, longitude: -0.12765);
      final destination =
          Landmark.withLatLng(latitude: 51.27483, longitude: 0.52316);

      waypoints.add(departure);
      waypoints.add(destination);
    }

    final routePreferences = RoutePreferences();

    _showSnackBar(context,
        message: isFirstMap
            ? 'The first route is calculating.'
            : 'The second route is calculating.');

    if (isFirstMap) {
      _routingHandler1 = RoutingService.calculateRoute(
          waypoints,
          routePreferences,
          (err, routes) => _onRouteBuiltFinished(err, routes, true));
    } else {
      _routingHandler2 = RoutingService.calculateRoute(
          waypoints,
          routePreferences,
          (err, routes) => _onRouteBuiltFinished(err, routes, false));
    }
  }

  void _onRouteBuiltFinished(
    GemError err,
    List<Route>? routes,
    bool isFirstMap,
  ) {
    // If the route calculation is finished, we don't have a progress listener anymore.
    if (isFirstMap) {
      _routingHandler1 = null;
    } else {
      _routingHandler2 = null;
    }

    ScaffoldMessenger.of(context).clearSnackBars();
    if (_routingHandler1 != null) {
      _showSnackBar(context, message: 'The first route is calculating.');
    }
    if (_routingHandler2 != null) {
      _showSnackBar(context, message: 'The second route is calculating.');
    }

    // If there aren't any errors, we display the routes.
    if (err == GemError.success) {
      // Get the routes collection from map preferences.
      final routesMap =
          (isFirstMap
                  ? _mapController1.preferences
                  : _mapController2.preferences)
              .routes;

      // Display the routes on map.
      for (final route in routes!) {
        routesMap.add(route, route == routes.first, label: route.getMapLabel());
      }

      // Center the camera on routes.
      if (isFirstMap) {
        _mapController1.centerOnRoutes(routes: routes);
      } else {
        _mapController2.centerOnRoutes(routes: routes);
      }
    }
  }

  void _removeRoutes() {
    if (_routingHandler1 != null) {
      RoutingService.cancelRoute(_routingHandler1!);
      _routingHandler1 = null;
    }

    if (_routingHandler2 != null) {
      RoutingService.cancelRoute(_routingHandler2!);
      _routingHandler2 = null;
    }

    _mapController1.preferences.routes.clear();
    _mapController2.preferences.routes.clear();
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


