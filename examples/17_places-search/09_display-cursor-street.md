---
description: Documentation for Display Cursor Street
title: Display Cursor Street
---

# Display Cursor Street Name

This example demonstrates how to create a Flutter app that displays the name of the street at the cursor position using Maps SDK for Flutter. When the user taps on the map, the app retrieves and displays the street name at that location.

## How It Works 

- Main App Setup : Initializes the Maps SDK and sets up the app’s home screen with a map.

- Displaying Street Name : The map detects touch input, centers on the selected coordinates, and displays the street name at the tapped location in a bottom-centered container.

### UI and Map Integration

The map screen includes an AppBar and displays a container at the bottom showing the street name upon tapping a location on the map. The container at the bottom displays the street name when it is available.
```dart
class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key});

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  late GemMapController _mapController;
  String _currentStreetName = "";

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
          'Display Cursor Street Name',
          style: TextStyle(color: Colors.white),
        ),
      ),
      body: Stack(
        alignment: AlignmentDirectional.bottomCenter,
        children: [
          GemMap(
            key: ValueKey("GemMap"),
            onMapCreated: _onMapCreated,
            appAuthorization: projectApiToken,
          ),
          if (_currentStreetName != "")
            Padding(
              padding: const EdgeInsets.symmetric(vertical: 25.0),
              child: Container(
                color: Colors.white,
                child: Padding(
                  padding: const EdgeInsets.all(8.0),
                  child: Text(_currentStreetName),
                ),
              ),
            ),
        ],
      ),
    );
  }
  
```

### Handling Cursor Position and Retrieving Street Name

This code sets the cursor to follow the tapped location and retrieves the street name. This code initializes the map controller, sets the cursor to follow the user’s taps, and retrieves the street name, which is then displayed in the UI.
```dart
void _onMapCreated(GemMapController controller) async {
  _mapController = controller;

  _mapController.centerOnCoordinates(
      Coordinates(latitude: 45.472358, longitude: 9.184945));

  _mapController.preferences.enableCursor = true;
  _mapController.preferences.enableCursorRender = true;

  // Register touch callback to set cursor to tapped position
  _mapController.registerTouchCallback((point) async {
    await _mapController.setCursorScreenPosition(point);
    final streets = _mapController.cursorSelectionStreets();
    setState(() {
      _currentStreetName = streets.isEmpty ? "Unnamed street" : streets.first.name;
    });
  });
}
```


