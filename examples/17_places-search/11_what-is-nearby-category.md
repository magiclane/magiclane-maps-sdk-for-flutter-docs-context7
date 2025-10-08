---
description: Documentation for What Is Nearby Category
title: What Is Nearby Category
---

# What Is Nearby Category

This example demonstrates how to create a Flutter app that shows nearby landmarks based on the user’s current position using the Maps SDK for Flutter.

## How it works

This example app demonstrates the following features:

- Display a `GemMap`.

- Get current position.

- Perform a search for nearby landmarks (gas stations category).

- Display search results.

### UI and Map Integration

The following code builds an UI with an interactive `GemMap`, an app bar with a button for navigating to search results page.
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
      title: 'What\'s Nearby Category',
      debugShowCheckedModeBanner: false,
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
        title: const Text(
          'What\'s Nearby Category',
          style: TextStyle(color: Colors.white),
        ),
        actions: [
          IconButton(
            onPressed: () => _onWhatIsNearbyButtonPressed(context),
            icon: const Icon(Icons.question_mark, color: Colors.white),
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

  // The callback for when map is ready to use.
  void _onMapCreated(GemMapController controller) async {
    // Save controller for further usage.
    _mapController = controller;

    if (kIsWeb) {
      // On web platform permission are handled differently than other platforms.
      // The SDK handles the request of permission for location.
      final locationPermissionWeb =
          await PositionService.requestLocationPermission();
      if (locationPermissionWeb == true) {
        _locationPermissionStatus = PermissionStatus.granted;
      } else {
        _locationPermissionStatus = PermissionStatus.denied;
      }
    } else {
      // For Android & iOS platforms, permission_handler package is used to ask for permissions.
      _locationPermissionStatus = await Permission.locationWhenInUse.request();
    }

    if (_locationPermissionStatus == PermissionStatus.granted) {
      // After the permission was granted, we can set the live data source (in most cases the GPS).
      // The data source should be set only once, otherwise we'll get -5 error.
      if (!_hasLiveDataSource) {
        PositionService.setLiveDataSource();
        _hasLiveDataSource = true;
      }

      // Optionally, we can set an animation
      final animation = GemAnimation(type: AnimationType.linear);

      // Calling the start following position SDK method.
      _mapController.startFollowingPosition(animation: animation);
    }
  }

  void _onWhatIsNearbyButtonPressed(BuildContext context) {
    // Get the current position with no altitude
    final currentPosition = PositionService.position;

    if (currentPosition == null) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('No position currently available')),
      );
      return;
    }

    final currentPositionNoAltitude = Coordinates(
      latitude: currentPosition.latitude,
      longitude: currentPosition.longitude,
      altitude: 0.0,
    );

    // Pass the current position
    Navigator.of(context).push(
      MaterialPageRoute<dynamic>(
        builder:
            (context) =>
                WhatIsNearbyCategoryPage(position: currentPositionNoAltitude),
      ),
    );
  }
}
```

### Search Results Page
```dart
class WhatIsNearbyCategoryPage extends StatefulWidget {
  final Coordinates position;
  const WhatIsNearbyCategoryPage({super.key, required this.position});

  @override
  State<WhatIsNearbyCategoryPage> createState() =>
      _WhatIsNearbyCategoryPageState();
}

class _WhatIsNearbyCategoryPageState extends State<WhatIsNearbyCategoryPage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        automaticallyImplyLeading: true,
        foregroundColor: Colors.white,
        title: const Text(
          "What's Nearby Category",
          style: TextStyle(color: Colors.white),
        ),
        backgroundColor: Colors.deepPurple[900],
      ),
      body: FutureBuilder(
        future: _getNearbyLocations(),
        builder: (context, snapshot) {
          if (!snapshot.hasData || snapshot.data == null) {
            return const Center(child: CircularProgressIndicator());
          }
          return ListView.separated(
            itemBuilder: (context, index) {
              return NearbyItem(
                landmark: snapshot.data!.elementAt(index),
                currentPosition: widget.position,
              );
            },
            separatorBuilder:
                (context, index) => const Divider(indent: 0, height: 0),
            itemCount: snapshot.data!.length,
          );
        },
      ),
    );
  }

  Future<List<Landmark>?> _getNearbyLocations() async {
    // Add the gas stations category to SearchPreferences
    final preferences = SearchPreferences(searchAddresses: false);
    final genericCategories = GenericCategories.categories;
    final gasStationCategory = genericCategories.firstWhere(
      (category) => category.name == 'Gas Stations',
    );

    preferences.landmarks.addStoreCategoryId(
      gasStationCategory.landmarkStoreId,
      gasStationCategory.id,
    );

    final completer = Completer<List<Landmark>?>();
    // Perform search around position with current position and preferences set with gas stations category
    SearchService.searchAroundPosition(
      preferences: preferences,
      widget.position,
      (err, result) {
        completer.complete(result);
      },
    );
    return completer.future;
  }
}

class NearbyItem extends StatefulWidget {
  final Landmark landmark;
  final Coordinates currentPosition;
  const NearbyItem({
    super.key,
    required this.landmark,
    required this.currentPosition,
  });

  @override
  State<NearbyItem> createState() => _NearbyItemState();
}

class _NearbyItemState extends State<NearbyItem> {
  @override
  Widget build(BuildContext context) {
    return ListTile(
      title: Text(
        widget.landmark.name,
        overflow: TextOverflow.fade,
        style: const TextStyle(
          color: Colors.black,
          fontSize: 14,
          fontWeight: FontWeight.w400,
        ),
        maxLines: 2,
      ),
      leading:
          widget.landmark.img.isValid
              ? Image.memory(
                widget.landmark.img.getRenderableImageBytes(
                  size: Size(128, 128),
                )!,
              )
              : SizedBox(),
      trailing: Text(
        _convertDistance(
          widget.landmark.coordinates.distance(widget.currentPosition).toInt(),
        ),
        overflow: TextOverflow.fade,
        style: const TextStyle(
          color: Colors.black,
          fontSize: 14,
          fontWeight: FontWeight.w400,
        ),
      ),
    );
  }

  String _convertDistance(int meters) {
    if (meters >= 1000) {
      double kilometers = meters / 1000;
      return '${kilometers.toStringAsFixed(1)} km';
    } else {
      return '${meters.toString()} m';
    }
  }
}
```


