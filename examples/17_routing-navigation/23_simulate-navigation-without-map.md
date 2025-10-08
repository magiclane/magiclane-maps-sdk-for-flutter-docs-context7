---
description: Documentation for Simulate Navigation Without Map
title: Simulate Navigation Without Map
---

# Simulate Navigation Without Map

In this guide, you will learn how to compute a route between a departure point and a destination point, and then simulate navigation along the route without a map.

## How it works

The example app highlights the following features:

- Calculate a route.

- Simulate navigation along a route.

### UI and Map Integration

The following code builds the UI with an app bar containing a build route button, and start and stop navigation buttons. Top and bottom navigation panels appear when navigating.
```dart
const projectApiToken = String.fromEnvironment('GEM_TOKEN');
Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await GemKit.initialize(appAuthorization: projectApiToken);
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Simulate Navigation Without Map',
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
  late NavigationInstruction _currentInstruction;

  // Store the computed route.
  late Route _route;

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
        title: const Text("Simulate Navigation Without Map", style: TextStyle(color: Colors.white)),
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
          if (_isSimulationActive)
            Positioned(top: 10, left: 10, child: BottomNavigationPanel(instruction: _currentInstruction)),
          if (_isSimulationActive)
            Positioned(
              bottom: MediaQuery.of(context).padding.bottom + 10,
              left: 0,
              child: BottomNavigationPanel(
                remainingDistance: getFormattedRemainingDistance(_currentInstruction),
                eta: getFormattedRemainingDistance(_currentInstruction),
                remainingDuration: getFormattedETA(_currentInstruction),
              ),
            ),
        ],
      ),
      resizeToAvoidBottomInset: false,
    );
  }

  // Custom method for calling calculate route and displaying the results.
  void _onBuildRouteButtonPressed(BuildContext context) {
    // Define the departure.
    final departureLandmark = Landmark.withLatLng(latitude: 51.20830988558932, longitude: 6.6794155000229045);

    // Define the destination.
    final destinationLandmark = Landmark.withLatLng(latitude: 50.93416933110433, longitude: 6.94370301382495);

    // Define the route preferences.
    final routePreferences = RoutePreferences();
    _showSnackBar(context, message: 'The route is calculating.');

    // Calling the calculateRoute SDK method.
    // (err, results) - is a callback function that gets called when the route computing is finished.
    // err is an error enum, results is a list of routes.
    _routingHandler = RoutingService.calculateRoute([departureLandmark, destinationLandmark], routePreferences, (
      err,
      routes,
    ) async {
      // If the route calculation is finished, we don't have a progress listener anymore.
      _routingHandler = null;

      ScaffoldMessenger.of(context).clearSnackBars();

      // If there aren't any errors, we display the routes.
      if (err == GemError.success) {
        _showSnackBar(context, message: 'Successfully calculated the route.', duration: const Duration(seconds: 2));
        setState(() {
          _route = routes.first;
        });
      }
      setState(() {
        _areRoutesBuilt = true;
      });
    });
  }
```


```dart
  // Method for starting the simulation and following the position,
  void _startSimulation() {
    _navigationHandler = NavigationService.startSimulation(
      _route,
      onNavigationInstruction: (instruction, events) {
        setState(() {
          _isSimulationActive = true;
        });
        _currentInstruction = instruction;
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
  }
```


```dart
// Method for removing the routes from display,
void _cancelRoute() {
  if (_routingHandler != null) {
    // Cancel the navigation.
    RoutingService.cancelRoute(_routingHandler!);
    _routingHandler = null;
  }

  setState(() {
    _areRoutesBuilt = false;
  });
}
```


```dart
// Method to stop the simulation and remove the displayed routes,
void _stopSimulation() {
  // Cancel the navigation.
  NavigationService.cancelNavigation(_navigationHandler!);
  _navigationHandler = null;

  _cancelRoute();

  setState(() => _isSimulationActive = false);
}
```


```dart
  // Method to show message in case calculate route is not finished,
  void _showSnackBar(BuildContext context, {required String message, Duration duration = const Duration(hours: 1)}) {
    final snackBar = SnackBar(content: Text(message), duration: duration);

    ScaffoldMessenger.of(context).showSnackBar(snackBar);
  }
```

### Top Navigation Panel
```dart
class BottomNavigationPanel extends StatelessWidget {
  final NavigationInstruction instruction;

  const BottomNavigationPanel({super.key, required this.instruction});

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
                  getFormattedDistanceToNextTurn(instruction),
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
class BottomNavigationPanel extends StatelessWidget {
  final String remainingDuration;
  final String remainingDistance;
  final String eta;

  const BottomNavigationPanel({
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


