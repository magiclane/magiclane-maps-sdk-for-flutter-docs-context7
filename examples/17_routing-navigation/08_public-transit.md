---
description: Documentation for Public Transit
title: Public Transit
---

# Public Transit

This example demonstrates how to create a Flutter app that displays public transit routes using Maps SDK for Flutter. Users can calculate and visualize routes between two landmarks.

## How it works 

The example app demonstrates the following features:

- Display a map.

- Calculate a public transport route between two coordinates.

- Display computed route on map along with transit segments at the bottom of the screen.

### UI and Map Integration

This code sets up the main screen with a map and functionality for calculating and displaying public transit routes.
```dart
void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Public Transit',
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

  // We use the handler to cancel the route calculation.
  TaskHandler? _routingHandler;

  List<PTRouteSegment>? _ptSegments;

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
          'Public Transit',
          style: TextStyle(color: Colors.white),
        ),
        actions: [
          // Routes are not built.
          if (_routingHandler == null && _ptSegments == null)
            IconButton(
              onPressed: () => _onBuildRouteButtonPressed(context),
              icon: const Icon(Icons.route, color: Colors.white),
            ),
          // Routes calculating is in progress.
          if (_routingHandler != null)
            IconButton(
              onPressed: () => _onCancelRouteButtonPressed(),
              icon: const Icon(Icons.stop, color: Colors.white),
            ),
          // Routes calculating is finished.
          if (_ptSegments != null)
            IconButton(
              onPressed: () => _onClearRoutesButtonPressed(),
              icon: const Icon(Icons.clear, color: Colors.white),
            ),
        ],
      ),
      body: Stack(
        alignment: AlignmentDirectional.bottomCenter,
        children: [
          GemMap(
            key: ValueKey("GemMap"),
            onMapCreated: _onMapCreated,
            appAuthorization: projectApiToken,
          ),
          if (_ptSegments != null)
            Padding(
              padding: const EdgeInsets.all(8.0),
              child: Container(
                height: MediaQuery.of(context).size.height * 0.1,
                width: MediaQuery.of(context).size.width * 0.9,
                color: Colors.white,
                child: Row(
                  // Build a TransitSegment to display data from each segment
                  children: _ptSegments!.map((segment) {
                    return TransitSegment(segment: segment);
                  }).toList(),
                ),
              ),
            ),
        ],
      ),
    );
  }

  // The callback for when map is ready to use.
  Future<void> _onMapCreated(GemMapController controller) async {
    // Save controller for further usage.
    _mapController = controller;

    // Register route tap gesture callback.
    await _registerRouteTapCallback();
  }
```

### Route Calculation

This code handles the route calculation and updates the UI with the calculated segments.
```dart
void _onBuildRouteButtonPressed(BuildContext context) {
  // Define the departure.
  final departureLandmark = Landmark.withLatLng(
    latitude: 51.505929,
    longitude: -0.097579,
  );

  // Define the destination.
  final destinationLandmark = Landmark.withLatLng(
    latitude: 51.507616,
    longitude: -0.105036,
  );

  // Define the route preferences with public transport mode.
  final routePreferences = RoutePreferences(
    transportMode: RouteTransportMode.public,
  );

  _showSnackBar(context, message: "The route is being calculated.");

  // Calling the calculateRoute SDK method.
  // (err, results) - is a callback function that gets called when the route computing is finished.
  // err is an error enum, results is a list of routes.

  _routingHandler = RoutingService.calculateRoute(
    [departureLandmark, destinationLandmark],
    routePreferences,
    (err, routes) {
      // If the route calculation is finished, we don't have a progress listener anymore.
      _routingHandler = null;
      ScaffoldMessenger.of(context).clearSnackBars();

      // If there aren't any errors, we display the routes.
      if (err == GemError.success) {
        // Get the routes collection from map preferences.
        final routesMap = _mapController.preferences.routes;

        // Display the routes on map.
        for (final route in routes) {
          routesMap.add(
            route,
            route == routes.first,
            label: route == routes.first ? getMapLabel(route) : null,
          );
        }

        // Center the camera on routes.
        _mapController.centerOnRoutes(routes: routes);
        // Convert normal route to PTRoute
        final ptRoute = routes.first.toPTRoute();
        // Convert each segment to PTRouteSegment
        final List<PTRouteSegment?> segments = ptRoute!.segments.map((seg) => seg.toPTRouteSegment()).toList();
        final List<PTRouteSegment> ptSegments = segments.where((seg) => seg != null).cast<PTRouteSegment>().toList();

        setState(() {
          _ptSegments = ptSegments;
        });
      }
    },
  );

  setState(() {});
}
```

### Transit Segment Display

This widget displays transit segments and relevant information about each route leg.
```dart
class TransitSegment extends StatelessWidget {
  final PTRouteSegment segment;

  const TransitSegment({super.key, required this.segment});

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.symmetric(horizontal: 8.0),
      child: segment.transitType == TransitType.walk
          ? Row(
              children: [
                const Icon(Icons.directions_walk, size: 35.0),
                Text(convertDuration(segment.timeDistance.totalDistanceM)),
              ],
            )
          : Row(
              children: [
                const Icon(Icons.directions_bus_outlined, size: 35.0),
                if (segment.hasWheelchairSupport) const Icon(Icons.accessible_forward),
                Container(
                  color: Colors.green,
                  child: Text(segment.shortName),
                ),
              ],
            ),
    );
  }
}
```

### Utility Functions

These utility functions convert distances and durations into user-friendly formats.
```dart
String convertDistance(int meters) {
  if (meters >= 1000) {
    double kilometers = meters / 1000;
    return '${kilometers.toStringAsFixed(1)} km';
  } else {
    return '${meters.toString()} m';
  }
}

// Utility function to convert the seconds duration into a suitable format
String convertDuration(int seconds) {
  int hours = seconds ~/ 3600;
  int minutes = (seconds % 3600) ~/ 60;
  int remainingSeconds = seconds % 60;

  String hoursText = (hours > 0) ? '$hours h ' : '';
  String minutesText = (minutes > 0) ? '$minutes min ' : '';
  String secondsText = (hours == 0 && minutes == 0) ? '$remainingSeconds sec' : '';

  return (hoursText + minutesText + secondsText).trim();
}
```

### Displaying Route Information
```dart
String getMapLabel(Route route) {
  // Get total distance and total duration from time distance.
  final totalDistance = route.getTimeDistance().unrestrictedDistanceM + route.getTimeDistance().restrictedDistanceM;
  final totalDuration = route.getTimeDistance().unrestrictedTimeS + route.getTimeDistance().restrictedTimeS;

  // Convert the route to a public transit route (PTRoute).
  final publicTransitRoute = route.toPTRoute();
  if (publicTransitRoute == null) {
    return "";
  }

  // Get the first and last segments of the route.
  final firstSegment = publicTransitRoute.segments.first.toPTRouteSegment();
  final lastSegment = publicTransitRoute.segments.last.toPTRouteSegment();

  if (firstSegment == null || lastSegment == null) {
    return "";
  }

  // Get departure and arrival times from the segments.
  final departureTime = firstSegment.departureTime;
  final arrivalTime = lastSegment.arrivalTime;

  // Calculate total walking distance (first and last segments are typically walking).
  final totalWalkingDistance = firstSegment.timeDistance.totalDistanceM + lastSegment.timeDistance.totalDistanceM;

  String formattedDepartureTime = "";
  String formattedArrivalTime = "";

  if (departureTime != null && arrivalTime != null) {
    // Format departure and arrival times.
    formattedDepartureTime = '${departureTime.hour}:${departureTime.minute.toString().padLeft(2, '0')}';
    formattedArrivalTime = '${arrivalTime.hour}:${arrivalTime.minute.toString().padLeft(2, '0')}';
  }

  // Build the label string with the route's details.
  return '${convertDuration(totalDuration)}\n' // Total duration
      '$formattedDepartureTime - $formattedArrivalTime\n' // Time range
      '${convertDistance(totalDistance)} ' // Total distance
      '(${convertDistance(totalWalkingDistance)} walking)\n' // Walking distance
      '${publicTransitRoute.publicTransportFare}'; // Fare
}
```


