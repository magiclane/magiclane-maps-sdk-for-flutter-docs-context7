---
description: Documentation for Search Along Route
title: Search Along Route
---

# Search Along Route

In this guide, you will learn how to calculate a route, simulate navigation, and search for landmarks along the route.

## How it works

This example demonstrates the following key features:

- Calculates a route between two landmarks and displays it on a map.

- Simulates navigation along the route, allowing you to start and stop navigation.

- Provides a feature to search for landmarks along the calculated route and displays the results.

### UI and Map Integration
```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Search Along Route',
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
  bool _isSimulationActive = false;
  bool _areRoutesBuilt = false;

  // We use the handler to cancel the route calculation.
  TaskHandler? _routingHandler;

  // We use the handler to cancel the navigation.
  TaskHandler? _navigationHandler;

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
        title: const Text("Search Along Route", style: TextStyle(color: Colors.white)),
        leading: Row(
          children: [
            if (_areRoutesBuilt)
              IconButton(
                onPressed: _searchAlongRoute,
                icon: const Icon(Icons.search, color: Colors.white),
              ),
          ],
        ),
        actions: [
          if (!_isSimulationActive && _areRoutesBuilt)
            IconButton(
              onPressed: _startSimulation,
              icon: const Icon(Icons.play_arrow, color: Colors.white),
            ),
          if (_isSimulationActive)
            IconButton(
              onPressed: _stopSimulation,
              icon: const Icon(
                Icons.stop,
                color: Colors.white,
              ),
            ),
          if (!_areRoutesBuilt)
            IconButton(
              onPressed: () => _onBuildRouteButtonPressed(),
              icon: const Icon(
                Icons.route,
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

  void _onMapCreated(GemMapController controller) {
    _mapController = controller;
}
```

### Route calculation

This section shows how to calculate a route between two landmarks and display it on the map.
```dart
Future<void> _onBuildRouteButtonPressed() async {
  // Define the departure.
  final departureLandmark = Landmark.withLatLng(latitude: 37.77903, longitude: -122.41991);

  // Define the destination.
  final destinationLandmark = Landmark.withLatLng(latitude: 37.33619, longitude: -121.89058);

  // Define the route preferences.
  final routePreferences = RoutePreferences();
  _showSnackBar(context, message: 'The route is calculating.');

  _routingHandler =
      RoutingService.calculateRoute([departureLandmark, destinationLandmark], routePreferences, (err, routes) async {
    // If the route calculation is finished, we don't have a progress listener anymore.
    _routingHandler = null;

    ScaffoldMessenger.of(context).clearSnackBars();

    // If there aren't any errors, we display the routes.
    if (err == GemError.success) {
      // Get the routes collection from map preferences.
      final routesMap = _mapController.preferences.routes;

      // Display the routes on map.
      for (final route in routes!) {
        routesMap.add(route, route == routes.first, label: route.getMapLabel());
      }

      _mapController.centerOnRoute(routes.first);
    }

    setState(() {
      _areRoutesBuilt = true;
    });
  });
}
```

### Navigation Simulation

This section demonstrates how to start and stop a simulated navigation along the calculated route.
```dart
void _startSimulation() {
  if (_isSimulationActive) return;
  if (!_areRoutesBuilt) return;

  _mapController.preferences.routes.clearAllButMainRoute();
  final routes = _mapController.preferences.routes;

  if (routes.mainRoute == null) {
    _showSnackBar(context, message: "No main route available");
    return;
  }

  _navigationHandler = NavigationService.startSimulation(
    routes.mainRoute!,
    null,
    onNavigationInstruction: (instruction, events) {
      setState(() {
        _isSimulationActive = true;
      });
    },
    onError: (error) {
      // If the navigation has ended or if and error occurred while navigating, remove routes.
      setState(() {
        _isSimulationActive = false;
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

  setState(() {
    _isSimulationActive = true;
  });
}

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

// Stop simulated navigation.
void _stopSimulation() {
  // Cancel the navigation.
  NavigationService.cancelNavigation(_navigationHandler!);
  _navigationHandler = null;

  _cancelRoute();

  setState(() {
    _isSimulationActive = false;
    _areRoutesBuilt = false;
  });
}
```

### Search Along Route

The following code shows how to search for landmarks along the calculated route. The search results are printed to the console.
```dart
void _searchAlongRoute() {
  if (!_areRoutesBuilt) return;

  final routes = _mapController.preferences.routes;

  if (routes.mainRoute == null) {
    _showSnackBar(context, message: "No main route available");
    return;
  }

  // Calling the search along route SDK method.
  // (err, results) - is a callback function that gets called when the search is finished.
  // err is an error enum, results is a list of landmarks.
  SearchService.searchAlongRoute(routes.mainRoute!, (err, results) {
    if (err != GemError.success) {
      print("SearchAlongRoute - no results found");
      return;
    }

    print("SearchAlongRoute - ${results.length} results:");
    for (final Landmark landmark in results) {
      final landmarkName = landmark.name;
      print("SearchAlongRoute: $landmarkName");
    }
  });
}
```

The result of the search operation is written in the console.

### Utility Functions

Utility functions are defined to show messages and format route labels.
```dart
  void _showSnackBar(
    BuildContext context, {
    required String message,
    Duration duration = const Duration(hours: 1),
  }) {
    final snackBar = SnackBar(content: Text(message), duration: duration);

    ScaffoldMessenger.of(context).showSnackBar(snackBar);
  }

// Define an extension for route for calculating the route label which will be displayed on map.
extension RouteExtension on Route {
  String getMapLabel() {
    final totalDistance =
        getTimeDistance().unrestrictedDistanceM +
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


