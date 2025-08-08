---
description: Documentation for Better Route Notification
title: Better Route Notification
---

# Better Route Notification

This guide will teach you how to get notified when a better route is detected during navigation.

## How It Works

The example app demonstrates the following key features:

- Rendering an interactive map.

- Calculating routes with enhanced detection for better alternatives.

- Simulating navigation along a predefined route.

- Providing detailed insights on newly identified routes.

The example functionality is highly dependent on current traffic conditions. If the time difference between the selected route and the others is no greater than 5 minutes, the notification will not appear. See the [Better Route Detection](/guides/navigation/better-route-detection) documentation.

### UI and Map Integration

The following code demonstrates how to create a user interface with a `GemMap` widget and an app bar. The app bar includes buttons for calculating a route and initiating simulated navigation along the longer route. When a better route is identified, a notification panel will appear at the bottom of the screen, awaiting dismissal.
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
      title: 'Better Route Notification',
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
  late NavigationInstruction currentInstruction;

  bool _areRoutesBuilt = false;
  bool _isSimulationActive = false;

  // We use the progress listener to cancel the route calculation.
  TaskHandler? _routingHandler;

  // We use the progress listener to cancel the navigation.
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
        title: const Text(
          "Better Route Notification",
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
          if (_isSimulationActive)
            Positioned(
              top: 10,
              left: 10,
              child: Column(
                spacing: 10,
                children: [
                  NavigationInstructionPanel(instruction: currentInstruction),
                  FollowPositionButton(
                    onTap: () => _mapController.startFollowingPosition(),
                  ),
                ],
              ),
            ),
          if (_isSimulationActive)
            Positioned(
              bottom: MediaQuery.of(context).padding.bottom + 10,
              left: 0,
              child: NavigationBottomPanel(
                remainingDistance: currentInstruction.getFormattedRemainingDistance(),
                eta: currentInstruction.getFormattedRemainingDuration(),
                remainingDuration: currentInstruction.getFormattedETA(),
              ),
            ),
        ],
      ),
      resizeToAvoidBottomInset: false,
    );
  }

  void _onMapCreated(GemMapController controller) {
    _mapController = controller;
  }

  // Custom method for calling calculate route and displaying the results.
  void _onBuildRouteButtonPressed(BuildContext context) {
    // Define the departure.
    final departureLandmark = Landmark.withLatLng(
      latitude: 48.79743778098061,
      longitude: 2.4029037044571875,
    );

    // Define the destination.
    final destinationLandmark = Landmark.withLatLng(
      latitude: 48.904767018940184,
      longitude: 2.3223936076132086,
    );

    // Define the route preferences.
    final routePreferences = RoutePreferences(
      routeType: RouteType.fastest,
      avoidTraffic: TrafficAvoidance.all,
      transportMode: RouteTransportMode.car,
    );
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
            routesMap.add(
              route,
              route == routes.first,
              label: route.getMapLabel(),
            );
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

    routes.mainRoute = routes.at(1);

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
        currentInstruction = instruction;
      },
      onBetterRouteDetected: (route, travelTime, delay, timeGain) {
        // Display notification when a better route is detected.
        showModalBottomSheet<VoidCallbackAction>(
          context: context,
          isScrollControlled: true,
          backgroundColor: Colors.transparent,
          builder: (_) => BetterRoutePanel(
            travelTime: Duration(seconds: travelTime),
            delay: Duration(seconds: delay),
            timeGain: Duration(seconds: timeGain),
            onDismiss: () => Navigator.of(context).pop(),
          ),
        );
      },
      onBetterRouteInvalidated: () {
        print("The previously found better route is no longer valid");
      },
      onBetterRouteRejected: (reason) {
        print("The check for better route failed with reason: $reason");
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

    // Clear route alternatives from map.
    _mapController.preferences.routes.clearAllButMainRoute();

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

    _cancelRoute();

    setState(() => _isSimulationActive = false);
  }

  // Method to show message in case calculate route is not finished,
  void _showSnackBar(
    BuildContext context, {
    required String message,
    Duration duration = const Duration(hours: 1),
  }) {
    final snackBar = SnackBar(content: Text(message), duration: duration);

    ScaffoldMessenger.of(context).showSnackBar(snackBar);
  }
}

class FollowPositionButton extends StatelessWidget {
  const FollowPositionButton({super.key, required this.onTap});

  final VoidCallback onTap;

  @override
  Widget build(BuildContext context) {
    return InkWell(
      onTap: onTap,
      child: Container(
        height: 50,
        padding: const EdgeInsets.symmetric(horizontal: 10),
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
        child: const Row(
          mainAxisAlignment: MainAxisAlignment.spaceBetween,
          children: [
            Icon(Icons.navigation),
            Text(
              'Recenter',
              style: TextStyle(
                color: Colors.black,
                fontSize: 16,
                fontWeight: FontWeight.w600,
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Better Route Panel
```dart
class BetterRoutePanel extends StatelessWidget {
  final Duration travelTime;
  final Duration delay;
  final Duration timeGain;
  final VoidCallback onDismiss;

  const BetterRoutePanel({
    super.key,
    required this.travelTime,
    required this.delay,
    required this.timeGain,
    required this.onDismiss,
  });

  @override
  Widget build(BuildContext context) {
    return Material(
      elevation: 6,
      borderRadius: const BorderRadius.vertical(top: Radius.circular(16), bottom: Radius.circular(16)),
      child: Container(
        width: MediaQuery.of(context).size.width - 20,
        padding: const EdgeInsets.all(16),
        decoration: const BoxDecoration(
          color: Colors.white,
          borderRadius: BorderRadius.vertical(top: Radius.circular(16), bottom: Radius.circular(16)),
        ),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              'Better Route Detected',
              style: Theme.of(context).textTheme.titleMedium?.copyWith(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 12),

            // Inline info row: Total travel time
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                const Text('Total travel time:', style: TextStyle(fontWeight: FontWeight.w500)),
                Text('${travelTime.inMinutes} min'),
              ],
            ),
            const SizedBox(height: 4),

            // Inline info row: Traffic delay
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                const Text('Traffic delay:', style: TextStyle(fontWeight: FontWeight.w500)),
                Text('${delay.inMinutes} min'),
              ],
            ),
            const SizedBox(height: 4),

            // Inline info row: Time gain
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                const Text('Time gain:', style: TextStyle(fontWeight: FontWeight.w500)),
                Text('${timeGain.inMinutes} min'),
              ],
            ),
            const SizedBox(height: 16),

            Align(
              alignment: Alignment.centerRight,
              child: TextButton.icon(
                onPressed: onDismiss,
                icon: const Icon(Icons.close),
                label: const Text('Dismiss'),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Top Navigation Panel
```dart
class NavigationInstructionPanel extends StatelessWidget {
  final NavigationInstruction instruction;

  const NavigationInstructionPanel({super.key, required this.instruction});

  @override
  Widget build(BuildContext context) {
    return Container(
      width: MediaQuery.of(context).size.width - 20,
      height: MediaQuery.of(context).size.height * 0.2,
      padding: const EdgeInsets.all(10),
      decoration: BoxDecoration(
        color: Colors.black,
        borderRadius: BorderRadius.circular(15),
      ),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.start,
        children: [
          Container(
            padding: const EdgeInsets.all(20),
            width: 100,
            child: instruction.nextTurnDetails != null && instruction.nextTurnDetails!.abstractGeometryImg.isValid
                ? Image.memory(
                    instruction.nextTurnDetails!.abstractGeometryImg.getRenderableImageBytes(
                      size: Size(200, 200),
                      format: ImageFileFormat.png,
                    )!,
                    gaplessPlayback: true,
                  )
                : const SizedBox(), // Empty widget
          ),
          SizedBox(
            width: MediaQuery.of(context).size.width - 150,
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              mainAxisAlignment: MainAxisAlignment.start,
              children: [
                Text(
                  instruction.getFormattedDistanceToNextTurn(),
                  textAlign: TextAlign.left,
                  style: const TextStyle(color: Colors.white, fontSize: 25, fontWeight: FontWeight.w600),
                  overflow: TextOverflow.ellipsis,
                ),
                Text(
                  instruction.nextStreetName,
                  style: const TextStyle(color: Colors.white, fontSize: 20, fontWeight: FontWeight.w600),
                  overflow: TextOverflow.ellipsis,
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

### Bottom Navigation Panel
```dart
class NavigationBottomPanel extends StatelessWidget {
  final String remainingDuration;
  final String remainingDistance;
  final String eta;

  const NavigationBottomPanel({
    super.key,
    required this.remainingDuration,
    required this.remainingDistance,
    required this.eta,
  });

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
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
          Text(
            remainingDuration,
            style: const TextStyle(
              color: Colors.black,
              fontSize: 24,
              fontWeight: FontWeight.w500,
            ),
          ),
          Text(
            eta,
            style: const TextStyle(
              color: Colors.black,
              fontSize: 24,
              fontWeight: FontWeight.w500,
            ),
          ),
          Text(
            remainingDistance,
            style: const TextStyle(
              color: Colors.black,
              fontSize: 24,
              fontWeight: FontWeight.w500,
            ),
          ),
        ],
      ),
    );
  }
}
```

### Utility Functions
```dart
// Utility function to convert the meters distance into a suitable format
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
  int hours = seconds ~/ 3600; // Number of whole hours
  int minutes = (seconds % 3600) ~/ 60; // Number of whole minutes

  String hoursText = (hours > 0) ? '$hours h ' : ''; // Hours text
  String minutesText = '$minutes min'; // Minutes text

  return hoursText + minutesText;
}

// Utility function to add the given additional time to current time
String getCurrentTime({
  int additionalHours = 0,
  int additionalMinutes = 0,
  int additionalSeconds = 0,
}) {
  var now = DateTime.now();
  var updatedTime = now.add(
    Duration(
      hours: additionalHours,
      minutes: additionalMinutes,
      seconds: additionalSeconds,
    ),
  );
  var formatter = DateFormat('HH:mm');
  return formatter.format(updatedTime);
}

// Utility function to convert a raw image in byte data
Future<Uint8List?> imageToUint8List(Image? image) async {
  if (image == null) return null;
  final byteData = await image.toByteData(format: ImageByteFormat.png);
  return byteData!.buffer.asUint8List();
}

// Define an extension for route for calculating the route label which will be displayed on map
extension RouteExtension on Route {
  String getMapLabel() {
    final totalDistance =
        getTimeDistance().unrestrictedDistanceM +
        getTimeDistance().restrictedDistanceM;
    final totalDuration =
        getTimeDistance().unrestrictedTimeS + getTimeDistance().restrictedTimeS;

    return '${convertDistance(totalDistance)} \n${convertDuration(totalDuration)}';
  }
}

// Define an extension for navigation instruction to calculate distance and duration
extension NavigationInstructionExtension on NavigationInstruction {
  String getFormattedDistanceToNextTurn() {
    final totalDistanceToTurn =
        timeDistanceToNextTurn.unrestrictedDistanceM +
        timeDistanceToNextTurn.restrictedDistanceM;
    return convertDistance(totalDistanceToTurn);
  }

  String getFormattedDurationToNextTurn() {
    final totalDurationToTurn =
        timeDistanceToNextTurn.unrestrictedTimeS +
        timeDistanceToNextTurn.restrictedTimeS;
    return convertDuration(totalDurationToTurn);
  }

  String getFormattedRemainingDistance() {
    final remainingDistance =
        remainingTravelTimeDistance.unrestrictedDistanceM +
        remainingTravelTimeDistance.restrictedDistanceM;
    return convertDistance(remainingDistance);
  }

  String getFormattedRemainingDuration() {
    final remainingDuration =
        remainingTravelTimeDistance.unrestrictedTimeS +
        remainingTravelTimeDistance.restrictedTimeS;
    return convertDuration(remainingDuration);
  }

  String getFormattedETA() {
    final remainingDuration =
        remainingTravelTimeDistance.unrestrictedTimeS +
        remainingTravelTimeDistance.restrictedTimeS;
    return getCurrentTime(additionalSeconds: remainingDuration);
  }
}
```


