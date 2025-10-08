---
description: Documentation for Lane Instruction
title: Lane Instruction
---

# Lane Instruction

This example demonstrates how to build a Flutter app that calculates routes and displays lane instructions using the Maps SDK for Flutter.

## How it works

The example app demonstrates the following features:

- Initializes GemKit and calculates a route between landmarks.

- Allows users to simulate navigation along the calculated route.

- Displays lane instructions with real-time lane guidance.

### UI and Map Integration

The following code demonstrates how to build a user interface featuring a `GemMap` widget and an app bar with a route and navigation button. 
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
      title: 'Lane Instructions',
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
          "Lane Instructions",
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
          GemMap(key: ValueKey("GemMap"), onMapCreated: _onMapCreated, appAuthorization: projectApiToken),
          if (_isSimulationActive && currentInstruction.laneImg.isValid)
            Align(
              alignment: Alignment.bottomCenter,
              child: Padding(
                padding: EdgeInsets.only(bottom: MediaQuery.of(context).padding.bottom + 40),
                child: Container(
                  color: Colors.deepPurple[900],
                  padding: const EdgeInsets.all(12.0),
                  margin: const EdgeInsets.all(8.0),
                  child: Image.memory(
                    currentInstruction.laneImg
                        .getRenderableImage(size: Size(100, 50), format: ImageFileFormat.png, allowResize: true)! 
                        .bytes,
                    gaplessPlayback: true,
                  ),
                ),
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
}
```

### Route Calculation and Simulation

The calculateRoute method calculates a route between two landmarks and displays it on the map.
```dart
void _onBuildRouteButtonPressed(BuildContext context) {
  final departureLandmark = Landmark.withLatLng(latitude: 45.649572, longitude: 25.628333);
  final destinationLandmark = Landmark.withLatLng(latitude: 44.4379187, longitude: 26.0122374);

  final routePreferences = RoutePreferences();
    _showSnackBar(context, message: 'The route is calculating.');

    _routingHandler = RoutingService.calculateRoute(
      [departureLandmark, destinationLandmark],
      routePreferences,
      (err, routes) async {
        _routingHandler = null;

        ScaffoldMessenger.of(context).clearSnackBars();

        if (err == GemError.success) {
          final routesMap = _mapController.preferences.routes;

          for (final route in routes) {
            routesMap.add(
              route,
              route == routes.first,
              label: route.getMapLabel(),
            );
          }

          _mapController.centerOnRoutes(routes: routes);
        }
        setState(() {
          _areRoutesBuilt = true;
        });
      },
    );
  }
```

### Navigation Simulation

The startSimulation method triggers a simulated navigation session, updating the UI with lane instructions.
```dart
void _startSimulation() {

  final routes = _mapController.preferences.routes;
  _mapController.preferences.routes.clearAllButMainRoute();

  if (routes.mainRoute == null) {
    _showSnackBar(context, message: "No main route available");
    return;
  }

    _navigationHandler = NavigationService.startSimulation(
      routes.mainRoute!,
      onNavigationInstruction: (instruction, events) {
        setState(() {
          _isSimulationActive = true;
        });
        currentInstruction = instruction;
      },
      onError: (error) {
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
  _mapController.startFollowingPosition();
}
```

### UI Components

Lane Instruction Display shows current lane instructions based on the navigation data.
```dart
Align(
  alignment: Alignment.bottomCenter,
  child: Padding(
    padding: EdgeInsets.only(
      bottom: MediaQuery.of(context).padding.bottom + 40,
    ),
    child: Padding(
      padding: const EdgeInsets.all(8.0),
      // Call laneImg on instruction
      child: currentInstruction.laneImg.isValid
          ? Image.memory(
              currentInstruction.laneImg
                  .getRenderableImage(size: Size(100, 50), format: ImageFileFormat.png, allowResize: true)! 
                  .bytes,
              gaplessPlayback: true,
            )
          : SizedBox(),
    ),
  ),
),
```


