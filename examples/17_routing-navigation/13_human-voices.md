---
description: Documentation for Human Voices
title: Human Voices
---

# Human voices

This example demonstrates the functionalities of the Maps SDK for Flutter, including route calculation, simulation of navigation, and voice instructions.

## How It Works

This example integrates several components to simulate navigation with voice instructions. Here’s a breakdown of the key functionalities:

- Calculate routes based on user-defined landmarks and route preferences.

- Simulate navigation along a calculated route with real-time text-to-speech (TTS) instructions.

- Display and center routes on the map, with the camera following the simulated position.

### Apply Human Voice

Before the navigation has started, select an available human voice to play the instructions.
```dart
// Get the available list of human voices
final voices = ContentStore.getLocalContentList(ContentType.humanVoice);

// Apply the first voice
SdkSettings.setVoiceByPath(voices.first.fileName);
```

### Route Calculation

The user can trigger route calculation, which involves defining landmarks and preferences, then using RoutingService.calculateRoute to compute the route.
```dart
void _onBuildRouteButtonPressed(BuildContext context) {
  _showSnackBar(context, message: 'The route is calculating.');

  // Define landmarks.
  final departureLandmark =
      Landmark.withLatLng(latitude: 48.87586, longitude: 2.30311);
  final intermediaryPointLandmark =
      Landmark.withLatLng(latitude: 48.87422, longitude: 2.29952);
  final destinationLandmark =
      Landmark.withLatLng(latitude: 48.87361, longitude: 2.29513);

  // Define the route preferences.
  final routePreferences = RoutePreferences();

  // Calculate the route.
  _routingHandler = RoutingService.calculateRoute(
      [departureLandmark, intermediaryPointLandmark, destinationLandmark],
      routePreferences, (err, routes) {
    _routingHandler = null;
    ScaffoldMessenger.of(context).clearSnackBars();

    if (err == GemError.success) {
      final routesMap = _mapController.preferences.routes;
      for (final route in routes!) {
        routesMap.add(route, route == routes.first,
            label: route.getMapLabel());
      }
      _mapController.centerOnRoutes(routes: routes);
      setState(() => _areRoutesBuilt = true);
    }
  });
}
```

### Simulated Navigation

Once the route is built, the user can start the navigation simulation. The simulation triggers instructions play using the included human voices.
```dart
void _startSimulation() {
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
    speedMultiplier: 20,
  );

  // Set auto play sound to true, so that the voice instructions will be played automatically
  SoundPlayingService.canPlaySounds = true;

  _mapController.startFollowingPosition();
}
```

The canPlaySounds flag controlls if the SDK should automatically play TTS instructions using the selected voice.

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


