---
description: Documentation for Map Gestures
title: Map Gestures
---

# Map Gestures

This example demonstrates how to create a Flutter app that enables map gesture interactions using Maps SDK for Flutter. Users can interact with the map using gestures such as touch, move, angle change, and long press.

## How it works

The example app demonstrates the following features:

- Main App Setup : The main app initializes GemKit and displays a map.

- Map Gesture Handlers : Various gesture callbacks are registered on the map to track user interactions, including touch, movement, angle changes, and long presses.

### UI and Map Integration
```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Map Gestures',
      home: MyHomePage(),
    );
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key});

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}
```

### Map Gesture interactions
```dart
class _MyHomePageState extends State<MyHomePage> {
  late GemMapController _mapController;

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
        title: const Text('Map Gestures', style: TextStyle(color: Colors.white)),
      ),
      GemMap(
        key: ValueKey("GemMap"),
        onMapCreated: _onMapCreated,
        appAuthorization: projectApiToken,
      ),
    );
  }

  void _onMapCreated(GemMapController controller) {
    _mapController = controller;

    _mapController.registerOnMapAngleUpdate((angle) {
      print("Gesture: onMapAngleUpdate $angle");
    });

    _mapController.registerTouchCallback((point) {
      print("Gesture: onTouch $point");
    });

    _mapController.registerMoveCallback((point1, point2) {
      print('Gesture: onMove from (${point1.x} ${point1.y}) to (${point2.x} ${point2.y})');
    });

    _mapController.registerOnLongPressCallback((point) {
      print('Gesture: onLongPress $point');
    });
  }
}
```

This code sets up the main screen with a map and registers gesture handlers to print updates to the console based on user interactions.
