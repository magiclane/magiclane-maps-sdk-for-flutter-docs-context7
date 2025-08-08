---
description: Documentation for External Position Source Navigation
title: External Position Source Navigation
---

# External Position Source Navigation

This example demonstrates how to create a Flutter app that utilizes external position sources for navigation on a map using Maps SDK for Flutter. The app allows users to navigate to a predefined destination while following the route on the map.

## How It Works

The example app demonstrates the following features:

- Initialize a map.

- Navigation using external position sources.

- Allows route building and starts navigation with real-time position updates.

### UI and Navigation Integration

This code sets up the user interface, including a map and navigation buttons.
```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'External Position Source Navigation',
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
  late DataSource _dataSource;
  bool _areRoutesBuilt = false;
  bool _isNavigationActive = false;
  bool _hasDataSource = false;
  TaskHandler? _routingHandler;
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
          "ExternalPositionNavigation",
          style: TextStyle(color: Colors.white),
        ),
        backgroundColor: Colors.deepPurple[900],
        actions: [
          if (!_isNavigationActive && _areRoutesBuilt)
            IconButton(
              onPressed: () => _startNavigation(),
              icon: const Icon(Icons.play_arrow, color: Colors.white),
            ),
          if (_isNavigationActive)
            IconButton(
              onPressed: _stopNavigation,
              icon: const Icon(Icons.stop, color: Colors.white),
            ),
          if (!_areRoutesBuilt && _hasDataSource)
            IconButton(
              onPressed: () => _onBuildRouteButtonPressed(context),
              icon: const Icon(Icons.route, color: Colors.white),
            ),
          if (!_isNavigationActive)
            IconButton(
              onPressed: _onFollowPositionButtonPressed,
              icon: const Icon(
                Icons.location_searching_sharp,
                color: Colors.white,
              ),
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
          if (_isNavigationActive)
            Positioned(
              top: 10,
              left: 10,
              child: Column(
                children: [
                  NavigationInstructionPanel(instruction: currentInstruction),
                  const SizedBox(height: 10),
                  FollowPositionButton(
                    onTap: () => _mapController.startFollowingPosition(),
                  ),
                ],
              ),
            ),
          if (_isNavigationActive)
            Positioned(
              bottom: MediaQuery.of(context).padding.bottom + 10,
              left: 0,
              child: NavigationBottomPanel(
                remainingDistance:
                    currentInstruction.getFormattedRemainingDistance(),
                remainingDuration:
                    currentInstruction.getFormattedRemainingDuration(),
                eta: currentInstruction.getFormattedETA(),
              ),
            ),
        ],
      ),
      resizeToAvoidBottomInset: false,
    );
  }
}
```

### Handling Navigation and External Position Data

This code handles building the route from a departure point to a destination, notifying the user when the calculation is in progress.
```dart
void _onBuildRouteButtonPressed(BuildContext context) {
  final departureLandmark =
      Landmark.withLatLng(latitude: 34.915646, longitude: -110.147933);
  final destinationLandmark =
      Landmark.withLatLng(latitude: 34.933105, longitude: -110.131363);
  final routePreferences = RoutePreferences();
  _showSnackBar(context, message: 'The route is calculating.');

  _routingHandler = RoutingService.calculateRoute(
      [departureLandmark, destinationLandmark], routePreferences,
      (err, routes) {
    _routingHandler = null;
    ScaffoldMessenger.of(context).clearSnackBars();

    if (err == GemError.routeTooLong) {
      print('The destination is too far from your current location. Change the coordinates of the destination.');
      return;
    }

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
}
```

### Starting Navigation

This method starts the navigation and sets the map to follow the user’s position.
```dart
Future<void> _startNavigation() async {
  final routes = _mapController.preferences.routes;

  if (routes.mainRoute == null) {
    _showSnackBar(context, message: "Route is not available");
    return;
  }
    _navigationHandler = NavigationService.startNavigation(
      routes.mainRoute!,
      null,
      onNavigationInstruction: (instruction, events) {
        setState(() {
          _isNavigationActive = true;
        });
        currentInstruction = instruction;
      },
      onError: (error) {
        PositionService.instance.removeDataSource();
        _dataSource.stop();
        setState(() {
          _isNavigationActive = false;

          _cancelRoute();
        });
        if (error != GemError.cancel) {
          _stopNavigation();
        }
        return;
      },
      onDestinationReached: (landmark) {
        PositionService.instance.removeDataSource();
        _dataSource.stop();
        setState(() {
          _isNavigationActive = false;

          _cancelRoute();
        });
        _stopNavigation();
        return;
      },
    );

  _mapController.startFollowingPosition();
  await _pushExternalPosition();
}
```

### Pushing External Position Data

This code manages the position data, updating the user’s location along the route at regular intervals.
```dart
Future<void> _pushExternalPosition() async {
  final route = _mapController.preferences.routes.mainRoute;
  final distance = route.getTimeDistance().totalDistanceM;
  Coordinates prevCoordinates = route.getCoordinateOnRoute(0);

  for (int currentDistance = 1;
      currentDistance <= distance;
      currentDistance += 1) {
    if (!_hasDataSource) return;

    if (currentDistance == distance) {
      _stopNavigation();
      return;
    }

    final currentCoordinates = route.getCoordinateOnRoute(currentDistance);
    await Future<void>.delayed(Duration(milliseconds: 25));

    _dataSource.pushData(
        SenseDataFactory.producePosition(
          acquisitionTime: DateTime.now().toUtc().millisecondsSinceEpoch,
          latitude: currentCoordinates.latitude,
          longitude: currentCoordinates.longitude,
          altitude: 0,
          heading: _getHeading(prevCoordinates, currentCoordinates),
          speed: 0,
        ),
      );
  
    prevCoordinates = currentCoordinates;
  }
}
```

### Top Navigation Instruction Panel
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
      decoration: BoxDecoration(color: Colors.black, borderRadius: BorderRadius.circular(15)),
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


