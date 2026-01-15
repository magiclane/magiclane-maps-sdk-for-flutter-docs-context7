---
description: Documentation for Round Trip
title: Round Trip
---

# Round Trip

This example demonstrates how to calculate a round trip route using the `RoutingService` in a Flutter application. A round trip starts and ends at the same location, allowing users to explore a route and return to their starting point.

## How it works

The example app demonstrates the following features:

- Calculate a route from a single starting point that returns to the same location.

- Display the route on a map.

- Provide options to cancel route calculation or clear the routes from the map.

### UI and Map Integration

This code sets up the basic structure of the app, including the map and the app bar. It also provides buttons in the app bar for building, canceling, and clearing routes.
```dart
const projectApiToken = String.fromEnvironment("GEM_TOKEN");

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(debugShowCheckedModeBanner: false, title: 'Round Trip',
     home: MyHomePage());
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
        title: const Text('Round Trip', style: TextStyle(color: Colors.white)),
        actions: [
          if (!_areRoutesBuilt && _routingHandler == null)
            IconButton(
              onPressed: () => _onBuildRouteButtonPressed(context),
              icon: const Icon(Icons.route, color: Colors.white),
            ),
          if (_routingHandler != null)
            IconButton(
              onPressed: _onCancelRouteButtonPressed,
              icon: const Icon(Icons.stop, color: Colors.white),
            ),
          if (_areRoutesBuilt)
            IconButton(
              onPressed: _onClearRoutesButtonPressed,
              icon: const Icon(Icons.clear, color: Colors.white),
            ),
        ],
      ),
      body: GemMap(key: const ValueKey("GemMap"),
      appAuthorization: projectApiToken, 
      onMapCreated: _onMapCreated),
    );
  }

  void _onMapCreated(GemMapController mapController) {
    _mapController = mapController;
    // Map is ready to use
  }

  void _onBuildRouteButtonPressed(BuildContext context) {
    // Define departure landmark in Amsterdam
    final departureLandmark = Landmark.withLatLng(latitude: 52.361947, longitude: 4.864486);

    // Define round trip preferences
    final tripPreferences = RoundTripParameters(range: 5000, rangeType: RangeType.distanceBased);

    // Define route preferences to include round trip parameters
    final routePreferences = RoutePreferences(
      transportMode: RouteTransportMode.bicycle,
      roundTripParameters: tripPreferences,
    );

    _showSnackBar(context, message: 'Calculating round trip route...');

    // Use only the departure landmark to calculate a round trip route
    _routingHandler = RoutingService.calculateRoute([departureLandmark], 
    routePreferences,
    (err, routes) {
      _routingHandler = null;
      ScaffoldMessenger.of(context).clearSnackBars();

      if (err == GemError.success && routes.isNotEmpty) {
        final routesMap = _mapController.preferences.routes;

        for (final route in routes) {
          routesMap.add(route, route == routes.first, label: getMapLabel(route));
        }

        _mapController.centerOnRoutes(routes: routes);

        setState(() {
          _areRoutesBuilt = true;
        });
      } else {
        _showSnackBar(context, message: 'Failed to calculate route',
         duration: const Duration(seconds: 3));
      }
    });

    setState(() {});
  }

  void _onClearRoutesButtonPressed() {
    _mapController.preferences.routes.clear();

    setState(() {
      _areRoutesBuilt = false;
    });
  }

  void _onCancelRouteButtonPressed() {
    if (_routingHandler != null) {
      RoutingService.cancelRoute(_routingHandler!);

      setState(() {
        _routingHandler = null;
      });
    }
  }

  void _showSnackBar(BuildContext context, {required String message, 
    Duration duration = const Duration(hours: 1)}) {
    final snackBar = SnackBar(content: Text(message), duration: duration);
    ScaffoldMessenger.of(context).showSnackBar(snackBar);
  }
}
```

### Utility Functions
```dart
import 'package:magiclane_maps_flutter/routing.dart' show Route;

String convertDistance(int meters) {
  if (meters >= 1000) {
    double kilometers = meters / 1000;
    return '${kilometers.toStringAsFixed(1)} km';
  } else {
    return '$meters m';
  }
}

String convertDuration(int seconds) {
  int hours = seconds ~/ 3600;
  int minutes = (seconds % 3600) ~/ 60;

  String hoursText = (hours > 0) ? '$hours h ' : '';
  String minutesText = '$minutes min';

  return hoursText + minutesText;
}

String getMapLabel(Route route) {
  return '${convertDistance(route.getTimeDistance().totalDistanceM)} \n${convertDuration(route.getTimeDistance().totalTimeS)}';
}
```

