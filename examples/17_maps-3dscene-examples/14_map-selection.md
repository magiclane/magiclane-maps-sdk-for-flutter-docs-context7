---
description: Documentation for Map Selection
title: Map Selection
---

# Map Selection

This example demonstrates how to explore various points of interest (POIs) and select destinations with ease.

## How it works

This example demonstrates the following features:

- Explore and select Points of Interest (POIs) on the map.

- Highlight and display detailed information about selected landmarks.

- Dynamically interact with map features through user taps.

### UI and Map Integration

The selection is highlighted when it is tapped. The taps are listened to by _registerLandmarkTapCallback , which is called in the _onMapCreated() callback, executed when the interactive map is initialized and ready for use.
```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Map Selection',
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

  Landmark? _focusedLandmark;

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
        title: const Text('Map Selection', style: TextStyle(color: Colors.white)),
      ),
      body: Stack(children: [
        GemMap(
          key: ValueKey("GemMap"),
          onMapCreated: _onMapCreated,
          appAuthorization: projectApiToken,
        ),
        if (_focusedLandmark != null)
          Align(
              alignment: Alignment.bottomCenter,
              child: LandmarkPanel(
                onCancelTap: _onCancelLandmarkPanelTap,
                landmark: _focusedLandmark!,
              ))
      ]),
      resizeToAvoidBottomInset: false,
    );
  }

  // The callback for when map is ready to use.
  void _onMapCreated(GemMapController controller) {
    // Save controller for further usage.
    _mapController = controller;

    // Listen for map landmark selection events.
    _registerLandmarkTapCallback();
  }

  void _registerLandmarkTapCallback() {
    _mapController.registerTouchCallback((pos) async {
      // Select the object at the tap position.
      _mapController.setCursorScreenPosition(pos);

      // Get the selected landmarks.
      final landmarks = _mapController.cursorSelectionLandmarks();

      // Check if there is a selected Landmark.
      if (landmarks.isNotEmpty) {
        // Highlight the selected landmark.
        _mapController.activateHighlight(landmarks);

        setState(() {
          _focusedLandmark = landmarks[0];
        });

        // Use the map controller to center on coordinates.
        _mapController.centerOnCoordinates(landmarks[0].coordinates);
      }
    });
  }

  void _onCancelLandmarkPanelTap() {
    // Remove landmark highlights from the map.
    _mapController.deactivateAllHighlights();

    setState(() {
      _focusedLandmark = null;
    });
  }
}
```


