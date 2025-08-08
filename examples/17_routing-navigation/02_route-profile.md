---
description: Documentation for Route Profile
title: Route Profile
---

# Route Profile

In this guide you will learn how to display a map, calculate routes between multiple points, and show a detailed route profile.

## How it Works

This example demonstrates the following key features:

- Calculates routes and renders them on the map.

- Display detailed route profile, including an elevation chart that visualizes the terrain along the route.

### UI and Map Integration

The following code demonstrates how to create a UI with a `GemMap` and an app bar featuring a "Build Route" button. After the route is calculated, a scrollable route profile panel appears at the bottom of the screen, along with a close button in the top-right corner of the app bar.   
```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Route Profile',
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
  Route? _focusedRoute;

  final LineAreaChartController _chartController = LineAreaChartController();

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
        title: const Text('Route Profile', style: TextStyle(color: Colors.white)),
        actions: [
          if (_routingHandler == null && _focusedRoute == null)
            IconButton(
              onPressed: () => _onBuildRouteButtonPressed(context),
              icon: const Icon(Icons.route, color: Colors.white),
            ),
          if (_routingHandler != null)
            IconButton(
              onPressed: () => _onCancelRouteButtonPressed(),
              icon: const Icon(Icons.stop, color: Colors.white),
            ),
          if (_focusedRoute != null)
            IconButton(
              onPressed: () => _onClearRoutesButtonPressed(),
              icon: const Icon(Icons.clear, color: Colors.white),
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
          if (_focusedRoute != null)
            Align(
                alignment: Alignment.bottomCenter,
                child: RouteProfilePanel(
                  route: _focusedRoute!,
                  mapController: _mapController,
                  chartController: _chartController,
                  centerOnRoute: () => _centerOnRoute([_focusedRoute!]),
                ))
        ],
      ),
    );
  }

  void _onMapCreated(GemMapController controller) {
    _mapController = controller;
    _registerRouteTapCallback();
  }

  void _onBuildRouteButtonPressed(BuildContext context) {
    final departureLandmark =
        Landmark.withLatLng(latitude: 46.59344, longitude: 7.91069);
    final destinationLandmark =
        Landmark.withLatLng(latitude: 46.55945, longitude: 7.89293);
    final routePreferences = RoutePreferences(
        buildTerrainProfile: const BuildTerrainProfile(enable: true),
        transportMode: RouteTransportMode.pedestrian);

    _showSnackBar(context, message: "The route is being calculated.");

    _routingHandler = RoutingService.calculateRoute(
        [departureLandmark, destinationLandmark], routePreferences,
        (err, routes) {
      _routingHandler = null;
      ScaffoldMessenger.of(context).clearSnackBars();

      if (err == GemError.success) {
        final routesMap = _mapController.preferences.routes;

        for (final route in routes!) {
          routesMap.add(route, route == routes.first,
              label: route.getMapLabel());
        }

        _centerOnRoute(routes);
        setState(() {
          _focusedRoute = routes.first;
        });
      }
    });

    setState(() {});
  }

  void _onClearRoutesButtonPressed() {
    _mapController.deactivateAllHighlights();
    _mapController.preferences.routes.clear();

    setState(() {
      _focusedRoute = null;
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

  void _registerRouteTapCallback() {
    _mapController.registerTouchCallback((pos) async {
      _mapController.setCursorScreenPosition(pos);
      final routes = _mapController.cursorSelectionRoutes();

      if (routes.isNotEmpty) {
        _mapController.preferences.routes.mainRoute = routes.first;

        if (_chartController.setCurrentHighlight != null) {
          _chartController.setCurrentHighlight!(0);
        }

        setState(() {
          _focusedRoute = routes.first;
        });

        _centerOnRoute([_focusedRoute!]);
      }
    });
  }

  void _centerOnRoute(List<Route> route) {
    const appbarHeight = 50;
    const padding = 20;

    _mapController.centerOnRoutes(route,
        screenRect: RectType(
          x: 0,
          y: (appbarHeight + padding * MediaQuery.of(context).devicePixelRatio)
              .toInt(),
          width: (MediaQuery.of(context).size.width *
                  MediaQuery.of(context).devicePixelRatio)
              .toInt(),
          height: ((MediaQuery.of(context).size.height / 2 -
                    appbarHeight -
                    2 * padding * MediaQuery.of(context).devicePixelRatio) *
                  MediaQuery.of(context).devicePixelRatio)
              .toInt(),
        ));
  }

  void _showSnackBar(BuildContext context,
      {required String message, Duration duration = const Duration(hours: 1)}) {
    final snackBar = SnackBar(
      content: Text(message),
      duration: duration,
    );

    ScaffoldMessenger.of(context).showSnackBar(snackBar);
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


