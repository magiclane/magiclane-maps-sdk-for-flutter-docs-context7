---
description: Documentation for Assets Map Styles
title: Assets Map Styles
---

# Assets Map Styles

This example showcases how to build a Flutter app featuring an interactive map with a custom style, seamlessly imported from the assets folder, using the Maps SDK for Flutter.

## How it works

The example app demonstrates the following features:

- Display a map.

- Loading and applying map styles from the app assets folder. 

### Add style to project

In the root directory of your project, create a new folder named assets. Specify the path to the assets folder in the pubspec.yaml file. Modify the file as follows:
```yaml
  flutter:
    uses-material-design: true

    assets:
      - assets/
```

Place a `.style` file inside the assets directory.

### UI and Map Integration

The following code builds an UI with a `GemMap` widget and an app bar with a set map style button. 
```dart
class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key});

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  late GemMapController _mapController;
  bool _isStyleLoaded = false;

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
          'Assets Map Style',
          style: TextStyle(color: Colors.white),
        ),
        actions: [
          if (!_isStyleLoaded)
            IconButton(
              onPressed: () => _applyStyle(),
              icon: Icon(Icons.map, color: Colors.white),
            ),
        ],
      ),
      body: GemMap(
        key: ValueKey("GemMap"),
        onMapCreated: _onMapCreated,
        appAuthorization: projectApiToken,
      ),
    );
  }

  void _onMapCreated(GemMapController controller) async {
    _mapController = controller;
  }
```

This code sets up the main screen with a map and a button that triggers the _applyStyle method to load a custom style file.

### Loading and Applying Map Styles 

This code loads the .style file as bytes, applies it to the map with a smooth transition, and centers the map on specified coordinates.
```dart
Future<void> _applyStyle() async {
  _showSnackBar(context, message: "The map style is loading.");

  await Future<void>.delayed(Duration(milliseconds: 250));

  final styleData = await _loadStyle();

  _mapController.preferences.setMapStyleByBuffer(
    styleData,
    smoothTransition: true,
  );

  setState(() {
    _isStyleLoaded = true;
  });

  if (mounted) {
    ScaffoldMessenger.of(context).hideCurrentSnackBar();
  }

  _mapController.centerOnCoordinates(
    Coordinates(latitude: 45, longitude: 20),
    zoomLevel: 25,
  );
}
```

### Loading the Style File 

This method reads the .style file from assets and returns the data as Uint8List bytes.
```dart
Future<Uint8List> _loadStyle() async {
  final data = await rootBundle.load('assets/Basic_1_Oldtime-1_21_656.style');
  return data.buffer.asUint8List();
}
```


