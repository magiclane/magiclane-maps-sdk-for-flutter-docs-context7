---
description: Documentation for Gpx Route
title: Gpx Route
---

# GPX Route

This example demonstrates how to calculate a route based on GPX data, render and center the route on an interactive map, and navigate along the route.

## How it works

The example app demonstrates the following features:

- Import GPX data to calculate and display routes.

- Render the calculated route on the map and center the camera to fit the entire route.

- Simulate navigation along the route.

- Display notifications and format route labels.

### Saving Assets

Before running the app, ensure that you save the necessary files (gpx files) into the assets directory.

Update your pubspec.yaml file to include these assets:
```yaml
flutter:
  assets:
    - assets/
```

### UI and Map Integration

This code sets up the basic structure of the app, including the map and the app bar. It also provides buttons in the app bar for drawing, building, canceling, and clearing routes.
```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(debugShowCheckedModeBanner: false, title: 'GPX Route', home: MyHomePage());
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
  bool _isGpxDataLoaded = false;
  bool _areRoutesBuilt = false;

  // We use the handler to cancel the navigation.
  TaskHandler? _navigationHandler;

  @override
  void initState() {
    _copyGpxToAppDocsDir();
    super.initState();
  }

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
        title: const Text("GPX Route", style: TextStyle(color: Colors.white)),
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
              onPressed: _importGPX,
              icon: const Icon(Icons.route, color: Colors.white),
            ),
        ],
      ),
      body: GemMap(key: ValueKey("GemMap"), onMapCreated: _onMapCreated, appAuthorization: projectApiToken),
    );
  }

  // The callback for when map is ready to use.
  void _onMapCreated(GemMapController controller) {
    // Save controller for further usage.
    _mapController = controller;
  }
```

### Copying the GPX File

This function copies the recorded_route.gpx file from the assets directory to the app’s documents directory:
```dart
Future<void> _copyGpxToAppDocsDir() async {
  if (!kIsWeb) {
    final docDirectory = await getApplicationDocumentsDirectory();
    final gpxFile = File('${docDirectory.path}/recorded_route.gpx');
    final fileBytes = await rootBundle.load('assets/recorded_route.gpx');
    final buffer = fileBytes.buffer;
    await gpxFile.writeAsBytes(buffer.asUint8List(fileBytes.offsetInBytes, fileBytes.lengthInBytes));
  }
}
```

The function ensures that the GPX file is available in the app’s documents directory, ready for use during runtime.

### Importing GPX Data and Calculating Routes

This function reads GPX data from the file, calculates the routes, and displays them on the map:
```dart
//Read GPX data from file, then calculate & show the routes on map
Future<void> _importGPX() async {
  _showSnackBar(context, message: 'The route is calculating.');

  List<Landmark> landmarkList = [];

  if (kIsWeb) {
    final fileBytes = await rootBundle.load('assets/recorded_route.gpx');
    final buffer = fileBytes.buffer;
    final pathData = buffer.asUint8List(fileBytes.offsetInBytes, fileBytes.lengthInBytes);

    // Process GPX data using your existing method
    final gemPath = Path.create(data: pathData, format: PathFileFormat.gpx);
    landmarkList = gemPath.toLandmarkList();
  } else {
    //Read file from app documents directory
    final docDirectory = await getApplicationDocumentsDirectory();
    final gpxFile = File('${docDirectory.path}/recorded_route.gpx');

    //Return if GPX file is not found
    if (!await gpxFile.exists()) {
      print('GPX file does not exist (${gpxFile.path})');
      return;
    }

    final bytes = await gpxFile.readAsBytes();
    final pathData = Uint8List.fromList(bytes);

    //Get landmarklist containing all GPX points from file.
    final gemPath = Path.create(data: pathData, format: PathFileFormat.gpx);
    landmarkList = gemPath.toLandmarkList();
  }

  print("GPX Landmarklist size: ${landmarkList.length}");

  // Define the route preferences.
  final routePreferences = RoutePreferences(transportMode: RouteTransportMode.bicycle);

  // Calling the calculateRoute SDK method.
  // (err, results) - is a callback function that gets called when the route computing is finished.
  // err is an error enum, results is a list of routes.
  RoutingService.calculateRoute(landmarkList, routePreferences, (err, routes) {
    ScaffoldMessenger.of(context).clearSnackBars();

    // If there aren't any errors, we display the routes.
    if (err == GemError.success) {
      // Get the routes collection from map preferences.
      final routesMap = _mapController.preferences.routes;

      // Display the routes on map.
      for (final route in routes) {
        // The first route is the main route
        routesMap.add(route, route == routes.first, label: getMapLabel(route));
      }

      // Center the camera on routes.
      _mapController.centerOnRoutes(routes: routes);

      setState(() {
        _areRoutesBuilt = true;
      });
    }
  });
  _isGpxDataLoaded = true;
}
```

When the “Calculate Route” button is pressed, this function reads the GPX file from the app’s documents directory and calculates a route based on the landmarks (waypoints) extracted from the GPX data.
The routes are displayed on the map and the camera is centered to ensure all routes fit within the viewport.

### Starting and Stopping the Simulation

The simulation can be started or stopped using the following functions:
```dart
void _startSimulation() {
  if (_isSimulationActive) return;
  if (!_isGpxDataLoaded) return;

  final routes = _mapController.preferences.routes;

  if (routes.mainRoute == null) {
    _showSnackBar(context, message: "No route available");
    return;
  }

  // Start navigation one the main route.
  _navigationHandler = NavigationService.startSimulation(routes.mainRoute!, speedMultiplier: 2);

  // Set the camera to follow position.
  _mapController.startFollowingPosition();

  setState(() => _isSimulationActive = true);
}

void _stopSimulation() {
  // Remove the routes from map.
  _mapController.preferences.routes.clear();

  setState(() => _areRoutesBuilt = false);

  if (_isSimulationActive) {
    // Cancel the navigation.
    NavigationService.cancelNavigation(_navigationHandler!);
    _navigationHandler = null;

    setState(() => _isSimulationActive = false);
  }
}
```

The _startSimulation function initiates the navigation simulation along the main route, with the camera following the simulated position. The _stopSimulation function stops the simulation and clears the routes from the map.

### Utility Functions

Utility functions are also included to display messages and format route labels:
```dart
  // Method to show message in case calculate route is not finished
  void _showSnackBar(BuildContext context, {required String message, Duration duration = const Duration(hours: 1)}) {
    final snackBar = SnackBar(content: Text(message), duration: duration);

    ScaffoldMessenger.of(context).showSnackBar(snackBar);
  }
```

The _showSnackBar method displays messages during route calculation.

### Displaying Route Information
```dart
String getMapLabel(Route route) {
  return '${convertDistance(route.getTimeDistance().totalDistanceM)} \n${convertDuration(route.getTimeDistance().totalTimeS)}';
}

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
```


