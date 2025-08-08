---
description: Documentation for Offline Routing
title: Offline Routing
---

# Offline Routing

In this guide, you will learn how to implement offline routing functionality using the Maps SDK for Flutter. This example demonstrates how to download a map for offline use, disable internet access, and calculate routes offline.

## How It Works

This example demonstrates the following features:

- Download specific maps (e.g., for “Andorra”) to enable offline functionality.

- Disable internet access after successful map download to enforce offline usage.

- Compute routes offline between predefined waypoints, using the downloaded map data.

### UI and Map Integration
```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
        title: 'Offline Routing',
        debugShowCheckedModeBanner: false,
        home: MyHomePage());
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key});

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  late GemMapController _mapController;

  bool _areRoutesBuilt = false;

  // We use the handler to cancel the route calculation.
  TaskHandler? _routingHandler;

  bool _isDownloaded = false;
  double _downloadProgress = 0;

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
          'Offline Routing',
          style: TextStyle(color: Colors.white),
        ),
        actions: [
          // Map is downloading.
          if (_isDownloaded == false && _downloadProgress != 0)
            Container(
              width: 20,
              height: 20,
              margin: const EdgeInsets.only(right: 10.0),
              child: const Center(
                child: CircularProgressIndicator(color: Colors.white),
              ),
            ),
          // Map is not downloaded.
          if (_isDownloaded == false && _downloadProgress == 0)
            IconButton(
              onPressed: () => _setOfflineMap(),
              icon: const Icon(Icons.download, color: Colors.white),
            ),
          // Routes are not built.
          if (_routingHandler == null &&
              _areRoutesBuilt == false &&
              _isDownloaded == true)
            IconButton(
              onPressed: () => _onBuildRouteButtonPressed(context),
              icon: const Icon(Icons.route, color: Colors.white),
            ),
          // Routes calculating is in progress.
          if (_routingHandler != null)
            IconButton(
              onPressed: () => _onCancelRouteButtonPressed(),
              icon: const Icon(Icons.stop, color: Colors.white),
            ),
          // Routes calculating is finished.
          if (_areRoutesBuilt == true)
            IconButton(
              onPressed: () => _onClearRoutesButtonPressed(),
              icon: const Icon(Icons.clear, color: Colors.white),
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
    // Save controller for further usage.
    _mapController = controller;

    SdkSettings.setAllowOffboardServiceOnExtraChargedNetwork(
        ServiceGroupType.contentService, true);
  }
```

### Define Route Calculation Logic

Implement methods to build routes based on predefined waypoints and manage the download of maps.
```dart
void _onBuildRouteButtonPressed(BuildContext context) {
  // Define the departure.
  final departureLandmark =
      Landmark.withLatLng(latitude: 42.49720, longitude: 1.50498);

  // Define the destination.
  final destinationLandmark =
      Landmark.withLatLng(latitude: 42.51003, longitude: 1.53400);

  // Define the route preferences.
  final routePreferences = RoutePreferences();

  _showSnackBar(context, message: "The route is being calculated.");

  // Calling the calculateRoute SDK method.
  _routingHandler = RoutingService.calculateRoute(
      [departureLandmark, destinationLandmark], routePreferences,
      (err, routes) {
    _routingHandler = null;
    ScaffoldMessenger.of(context).clearSnackBars();

    if (err == GemError.success) {
      final routesMap = _mapController.preferences.routes;
      for (final route in routes!) {
        routesMap.add(route, route == routes.first,
            label: route.getMapLabel());
      }
      _mapController.centerOnRoutes(routes);
      setState(() {
        _areRoutesBuilt = true;
      });
    }
  });

  setState(() {});
}

void _onClearRoutesButtonPressed() {
  _mapController.preferences.routes.clear();
  setState(() {
    _areRoutesBuilt = false;
  });
}

void _onCancelRouteButtonPressed() {
  if (_routingHandler != null) {
    RoutingService.cancelRoute(_routingHandler!);
    setState(() {
      _routingHandler = null;
    });
  }
}
```

### Define Map Downloading Logic

Implement methods for downloading and managing the offline map.
```dart
Future<List<ContentStoreItem>> _getMaps() async {
  Completer<List<ContentStoreItem>> mapsList =
      Completer<List<ContentStoreItem>>();

  ContentStore.asyncGetStoreContentList(ContentType.roadMap,
      (err, items, isCached) {
    if (err == GemError.success && items != null) {
      mapsList.complete(items);
    }
  });
  return mapsList.future;
}

void _setOfflineMap() {
  final localMaps = ContentStore.getLocalContentList(ContentType.roadMap);

  if (localMaps.where((map) => map.name == 'Andorra').isNotEmpty) {
    setState(() {
      _isDownloaded = true;
    });

    SdkSettings.setAllowInternetConnection(false);
    return;
  }

  _getMaps().then((maps) {
    _downloadProgress = maps[4].downloadProgress.toDouble();
    _downloadMap(maps[4]);
  });
}

void _downloadMap(ContentStoreItem map) {
  map.asyncDownload(_onMapDownloadFinished,
      onProgressCallback: _onMapDownloadProgressUpdated,
      allowChargedNetworks: true);
}

void _onMapDownloadProgressUpdated(int progress) {
  setState(() => _downloadProgress = progress.toDouble());
}

void _onMapDownloadFinished(GemError err) {
    // If there is no error, we change the state
    if (err == GemError.success) {
      // Deny internet connection
      SdkSettings.setAllowInternetConnection(false);

      setState(() => _isDownloaded = true);
    }
  }
```

### Show SnackBar for User Feedback

Implement a method to show a SnackBar for providing feedback to the user.
```dart
void _showSnackBar(BuildContext context,
    {required String message, Duration duration = const Duration(hours: 1)}) {
  final snackBar = SnackBar(
    content: Text(message),
    duration: duration,
  );

  ScaffoldMessenger.of(context).showSnackBar(snackBar);
}
```

### Define Extension for Route Label

Define an extension to format the route label displayed on the map.
```dart
extension RouteExtension on Route {
  String getMapLabel() {
    final totalDistance = getTimeDistance().unrestrictedDistanceM +
        getTimeDistance().restrictedDistanceM;
    final totalDuration =
        getTimeDistance().unrestrictedTimeS + getTimeDistance().restrictedTimeS;

    return '${_convertDistance(totalDistance)} \n${_convertDuration(totalDuration)}';
  }

  String _convertDistance(int meters) {
    if (meters >= 1000) {
      double kilometers = meters / 1000;
      return '${kilometers.toStringAsFixed(1)} km';
    }
    return '$meters m';
  }

  String _convertDuration(int seconds) {
    if (seconds >= 3600) {
      int hours = seconds ~/ 3600;
      int minutes = (seconds % 3600) ~/ 60;
      return '${hours}h ${minutes}m';
    } else if (seconds >= 60) {
      int minutes = seconds ~/ 60;
      return '${minutes}m';
    }
    return '${seconds}s';
  }
}
```


