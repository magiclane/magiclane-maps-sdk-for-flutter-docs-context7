---
description: Documentation for Gpx Routing Thumbnail Image
title: Gpx Routing Thumbnail Image
---

# GPX Routing Thumbnail Image

This example demonstrates how to build a Flutter app using the Maps SDK to calculate a route from a GPX file, capture a screenshot of the displayed route, and show it on the screen.

## Saving Assets

Before running the app, ensure that you save the necessary file (a `.gpx` file) into the assets directory.

Update your `pubspec.yaml` file to include these assets:
```yaml
flutter:
  assets:
    - assets/
```

## How it works

The example app highlights the following features:

- Importing a GPX file from assets.

- Creating a path out of GPX data.

- Calculating a route from the path.

- Taking and displaying a screenshot of the computed route.

### UI and Map Integration

The following code creates a UI with an empty page and an app bar that includes an import button for the GPX file. Once the GPX file is imported, the route is calculated and displayed on a hidden `GemMap`. A screenshot of the route is then captured and displayed on the screen.
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
      title: 'GPX Routing Thumbnail Image',
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

  Uint8List? _screenshotImage;

  @override
  void initState() {
    _copyGpxToAppDocsDir();

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
        backgroundColor: Colors.deepPurple[900],
        title: const Text(
          "GPX Routing Thumbnail Image",
          style: TextStyle(color: Colors.white),
        ),
        actions: [
          if (_screenshotImage == null)
            IconButton(
              onPressed: _importGPX,
              icon: const Icon(Icons.download, color: Colors.white),
            ),
        ],
      ),
      body: Stack(
        children: [
          GemMap(
            appAuthorization: projectApiToken,
            onMapCreated: (controller) {
              _mapController = controller;
            },
          ),
          Positioned.fill(child: Container(color: Colors.white)),
          _screenshotImage != null
              ? Center(
                child: Image.memory(
                  _screenshotImage!,
                  width: MediaQuery.of(context).size.width - 100,
                  height: 500,
                ),
              )
              : const SizedBox(),
        ],
      ),
    );
  }

  //Read GPX data from file, then compute & show path on map
  Future<void> _importGPX() async {
    _showSnackBar(
      context,
      message: 'Importing GPX.',
      duration: Duration(seconds: 3),
    );

    Path gemPath;

    if (kIsWeb) {
      final imageBytes = await rootBundle.load('assets/recorded_route.gpx');
      final buffer = imageBytes.buffer;
      final pathData = buffer.asUint8List(
        imageBytes.offsetInBytes,
        imageBytes.lengthInBytes,
      );

      // Process GPX data using your existing method
      gemPath = Path.create(data: pathData, format: PathFileFormat.gpx);
    } else {
      //Read file from app documents directory
      final docDirectory = await getApplicationDocumentsDirectory();
      final gpxFile = File('${docDirectory.path}/recorded_route.gpx');

      //Return if GPX file is not found
      if (!await gpxFile.exists()) {
        print('GPX file does not exist (${gpxFile.path})');
        return;
      }

      final bytes = await gpxFile.readAsBytes();
      final pathData = Uint8List.fromList(bytes);

      //Get the Path entity containing all GPX points from file.
      gemPath = Path.create(data: pathData, format: PathFileFormat.gpx);

      final route = await _calculateRouteFromPath(gemPath);

      _presentRouteOnMap(route);

      // Center on path's area with margins
      _mapController.centerOnAreaRect(
        route.geographicArea,
        zoomLevel: 70,
        viewRc: Rectangle<int>(
          _mapController.viewport.width ~/ 3,
          _mapController.viewport.height ~/ 3,
          _mapController.viewport.width ~/ 3,
          _mapController.viewport.height ~/ 3,
        ),
      );

      // Wait for the map actions to complete
      await Future<void>.delayed(Duration(milliseconds: 500));

      // Capture the thumbnail image
      Uint8List? screenshotImage = await _mapController.captureImage();

      if (screenshotImage == null) {
        print("Error while taking screenshot.\n");
        return;
      }

      setState(() {
        _screenshotImage = screenshotImage;
      });
    }
  }

  void _presentRouteOnMap(Route route) {
    _mapController.preferences.routes.add(
      route,
      true,
      routeRenderSettings: RouteRenderSettings(
        options: {RouteRenderOptions.main, RouteRenderOptions.showWaypoints},
      ),
    );
  }

  Future<Route> _calculateRouteFromPath(Path path) {
    final routeCompleter = Completer<Route>();

    final waypoints = path.landmarkList;

    RoutingService.calculateRoute(
      waypoints,
      RoutePreferences(transportMode: RouteTransportMode.pedestrian),
      (err, routes) {
        if (err != GemError.success) {
          _showSnackBar(context, message: "Error while computing route.");
          return;
        }
        routeCompleter.complete(routes.first);
      },
    );
    return routeCompleter.future;
  }

  //Copy the recorded_route.gpx file from assets directory to app documents directory
  Future<void> _copyGpxToAppDocsDir() async {
    if (!kIsWeb) {
      final docDirectory = await getApplicationDocumentsDirectory();
      final gpxFile = File('${docDirectory.path}/recorded_route.gpx');
      final fileBytes = await rootBundle.load('assets/recorded_route.gpx');
      final buffer = fileBytes.buffer;
      await gpxFile.writeAsBytes(
        buffer.asUint8List(fileBytes.offsetInBytes, fileBytes.lengthInBytes),
      );
    }
  }

  // Method to show message in case calculate route is not finished
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


