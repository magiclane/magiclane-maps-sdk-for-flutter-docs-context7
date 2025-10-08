---
description: Documentation for Follow Position
title: Follow Position
---

# Follow Position

This example demonstrates how to create a Flutter app that follows the device’s location on a map using Maps SDK for Flutter, with an option to request location permissions if necessary.

## How it works

The example app demonstrates the following features:

- Requesting location permissions on Android and iOS, with automatic handling on web platforms.

- Setting the live data source for the map (typically the device’s GPS).

- Following the device’s location on the map with optional animation.

### UI and Map Integration
```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Follow Position',
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

  PermissionStatus _locationPermissionStatus = PermissionStatus.denied;
  bool _hasLiveDataSource = false;

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
        title: const Text('Follow Position', style: TextStyle(color: Colors.white)),
        actions: [
          IconButton(
            onPressed: _onFollowPositionButtonPressed,
            icon: const Icon(Icons.location_searching_sharp, color: Colors.white),
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
```

This code sets up the app’s user interface, including a map and a button to follow the device’s position.

### Handling Location Permissions and Following Position
```dart
// The callback for when the map is ready to use.
void _onMapCreated(GemMapController controller) async {
  // Save controller for further usage.
  _mapController = controller;
}

void _onFollowPositionButtonPressed() async {
  if (kIsWeb) {
    // On web platforms, permissions are handled differently.
    _locationPermissionStatus = PermissionStatus.granted;
  } else {
    // Request location permission on Android and iOS platforms.
    _locationPermissionStatus = await Permission.locationWhenInUse.request();
  }

  if (_locationPermissionStatus == PermissionStatus.granted) {
    // Set the live data source (GPS) if not already set.
    if (!_hasLiveDataSource) {
      PositionService.setLiveDataSource();
      _hasLiveDataSource = true;
    }

    // Optionally, set an animation.
    final animation = GemAnimation(type: AnimationType.linear);

    // Start following the device's position.
    _mapController.startFollowingPosition(animation: animation);

    setState(() {});
  }
}
```

This code handles the process of requesting location permissions, setting the GPS as the live data source, and starting the map’s follow position mode.

### Displaying Position Information 

When the “Follow Position” button is pressed, the _onFollowPositionButtonPressed() callback is triggered. It requests location permission if it hasn’t been granted already, and sets the location data source to the device’s GPS sensor.
Once the permission is granted and the data source is set, the map will follow the device’s location, centering the camera on the current position. The follow position feature is disabled if the user interacts with the map, such as by panning, until the button is pressed again.
