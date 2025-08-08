---
description: Documentation for Speed Tts Warning
title: Speed Tts Warning
---

# Speed Text-To-Speech Warning

This guide will teach you how to navigate a route while receiving audio alerts when approaching a speed limit overlay.

## How It Works

The example app demonstrates the following key features:

- Route calculation.

- Simulated navigation along a route.

- Playing audio notifications for changing speed limits.

### UI and Map Integration

The following code builds the UI with an app bar containing a build route button, start and stop navigation buttons. A bottom panel appears with the remaining current speed limit. When the speed limit changes, a human voice announces the new speed limit value.
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
      title: 'Speed Tts Warning',
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
  late TTSEngine _ttsEngine;

  bool _areRoutesBuilt = false;
  bool _isSimulationActive = false;

  // We use the progress listener to cancel the route calculation.
  TaskHandler? _routingHandler;

  // We use the progress listener to cancel the navigation.
  TaskHandler? _navigationHandler;
  // ignore: unused_field
  AlarmService? _alarmService;
  AlarmListener? _alarmListener;

  // The current speed
  int? _currentSpeed;

  @override
  void initState() {
    _ttsEngine = TTSEngine();
    _ttsEngine.initTts();

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
        title: const Text(
          "Speed Tts Warning",
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
          if (_currentSpeed != null)
            Positioned(
              bottom: MediaQuery.of(context).padding.bottom + 10,
              left: 0,
              child: BottomSpeedLimitPanel(speed: _currentSpeed!),
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
      latitude: 51.35416637819253,
      longitude: 9.378580176120199,
    );
    // Define the destination.
    final destinationLandmark = Landmark.withLatLng(
      latitude: 51.36704970265849,
      longitude: 9.404698019844462,
    );
    // Define the route preferences.
    final routePreferences = RoutePreferences();
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
            routesMap.add(route, route == routes.first);
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

    _mapController.preferences.routes.clearAllButMainRoute();

    if (routes.mainRoute == null) {
      _showSnackBar(context, message: "No main route available");
      return;
    }

    _alarmListener = AlarmListener(
      onSpeedLimit: (speed, limit, insideCityArea) async {
        final speedLimitConverted = (limit * 3.6).toInt();

        if (_currentSpeed != speedLimitConverted) {
          setState(() {
            _currentSpeed = speedLimitConverted;
          });

          final speedWarning = "Current speed limit: $speedLimitConverted";
          await _ttsEngine.speakText(speedWarning);
        }
      },
    );

    // Set the alarms service with the listener
    setState(() {
      _alarmService = AlarmService(_alarmListener!);
    });

    _navigationHandler = NavigationService.startSimulation(
      routes.mainRoute!,
      null,
      onNavigationInstruction: (instruction, events) {
        setState(() {
          _isSimulationActive = true;
        });
      },
      onDestinationReached: (landmark) {
        _stopSimulation();
        _cancelRoute();
      },
      onError: (error) {
        // If the navigation has ended or if and error occurred while navigating, remove routes and reset closest alarm.
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

    setState(() {
      _isSimulationActive = false;
      _currentSpeed = null;
      _alarmService = null;
    });
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
```

### Text-To-Speech Engine Integration

The app requires the `flutter_tts` package.
```dart
import 'package:flutter_tts/flutter_tts.dart';

enum TtsState { playing, stopped, paused, continued }

class TTSEngine {
  late FlutterTts flutterTts;
  String? language;
  String? engine;
  double volume = 0.5;
  double pitch = 1.0;
  double rate = 0.5;
  bool isCurrentLanguageInstalled = false;

  TtsState ttsState = TtsState.stopped;

  bool get isPlaying => ttsState == TtsState.playing;
  bool get isStopped => ttsState == TtsState.stopped;
  bool get isPaused => ttsState == TtsState.paused;
  bool get isContinued => ttsState == TtsState.continued;

  bool get isIOS => !kIsWeb && Platform.isIOS;
  bool get isAndroid => !kIsWeb && Platform.isAndroid;
  bool get isWindows => !kIsWeb && Platform.isWindows;
  bool get isWeb => kIsWeb;

  void initTts() {
    flutterTts = FlutterTts();

    _setAwaitOptions();

    if (isAndroid) {
      _getDefaultEngine();
      _getDefaultVoice();
    }

    if (kIsWeb) {
      rate = 0.75;
    }

    flutterTts.setStartHandler(() {
      ttsState = TtsState.playing;
    });

    flutterTts.setCompletionHandler(() {
      ttsState = TtsState.stopped;
    });

    flutterTts.setCancelHandler(() {
      ttsState = TtsState.stopped;
    });

    flutterTts.setPauseHandler(() {
      ttsState = TtsState.paused;
    });

    flutterTts.setContinueHandler(() {
      ttsState = TtsState.continued;
    });

    flutterTts.setErrorHandler((msg) {
      ttsState = TtsState.stopped;
    });
  }

  Future<void> _getDefaultEngine() async {
    await flutterTts.getDefaultEngine;
  }

  Future<void> _getDefaultVoice() async {
    await flutterTts.getDefaultVoice;
  }

  Future<void> setVolume(double volume) async {
    await flutterTts.setVolume(volume);
  }

  Future<void> speakText(String text) async {
    await flutterTts.setSpeechRate(rate);
    await flutterTts.setPitch(pitch);
    await flutterTts.speak(text);
  }

  Future<void> _setAwaitOptions() async {
    await flutterTts.awaitSpeakCompletion(true);
  }

  void dispose() {
    flutterTts.stop();
  }
}
```

### Bottom Speed Limit Panel
```dart
class BottomSpeedLimitPanel extends StatelessWidget {
  final int speed;

  const BottomSpeedLimitPanel({super.key, required this.speed});

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
        children: [
          Text(
            "Current speed limit: $speed km/h",
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


