---
description: Documentation for Projections
title: Projections
---

# Projections

This example showcases how to build a Flutter app featuring an interactive map. Users can select a point on map and see its coordinates in different projections systems.

## How it works

The example app includes the following features:

- Display a map.

- Select a point on the map.

- Display the coordinates of the selected point in different projection systems.

### UI and Map Integration

The code below builds a user interface featuring an interactive `GemMap` which can be tapped to select a point. When a point is selected, the coordinates are displayed in different projection systems.
```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      title: 'Projections',
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

  WGS84Projection? _wgsProjection;
  MGRSProjection? _mgrsProjection;
  UTMProjection? _utmProjection;
  LAMProjection? _lamProjection;
  W3WProjection? _w3wProjection;
  GKProjection? _gkProjection;
  BNGProjection? _bngProjection;

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
          'Projections',
          style: TextStyle(color: Colors.white),
        ),
      ),
      body: Stack(
        alignment: AlignmentDirectional.bottomStart,
        children: [
          GemMap(
            key: ValueKey("GemMap"),
            onMapCreated: _onMapCreated,
            appAuthorization: projectApiToken,
          ),
          if (_wgsProjection != null)
            ProjectionsPanel(
              wgsProjection: _wgsProjection,
              mgrsProjection: _mgrsProjection,
              utmProjection: _utmProjection,
              lamProjection: _lamProjection,
              w3wProjection: _w3wProjection,
              gkProjection: _gkProjection,
              bngProjection: _bngProjection,
              onClose: () {
                setState(() {
                  _wgsProjection = null;
                  _mgrsProjection = null;
                  _utmProjection = null;
                  _lamProjection = null;
                  _w3wProjection = null;
                  _gkProjection = null;
                  _bngProjection = null;
                });
              },
            ),
        ],
      ),
    );
  }

  // The callback for when map is ready to use.
  void _onMapCreated(GemMapController controller) async {
    // Save controller for further usage.
    _mapController = controller;

    _mapController.centerOnCoordinates(
      Coordinates(latitude: 45.472358, longitude: 9.184945),
      zoomLevel: 80,
    );

    // Enable cursor to render on screen
    _mapController.preferences.enableCursor = true;
    _mapController.preferences.enableCursorRender = true;

    // Register touch callback to set cursor to tapped position
    _mapController.registerOnTouch((point) async {
      // Transform the screen point to Coordinates
      final coords = _mapController.transformScreenToWgs(point);

      // Update cursor position on the map
      _mapController.setCursorScreenPosition(point);

      // Build WGS84 projection from Coordinates
      final wgsProjection = WGS84Projection(coords);

      final utmProjection = await convertProjection(wgsProjection, ProjectionType.utm) as UTMProjection?;
      final mgrsProjection = await convertProjection(wgsProjection, ProjectionType.mgrs) as MGRSProjection?;
      final lamProjection = await convertProjection(wgsProjection, ProjectionType.lam) as LAMProjection?;
      final w3wProjection = await convertProjection(wgsProjection, ProjectionType.w3w) as W3WProjection?;
      final gkProjection = await convertProjection(wgsProjection, ProjectionType.gk) as GKProjection?;
      final bngProjection = await convertProjection(wgsProjection, ProjectionType.bng) as BNGProjection?;

      setState(() {
        _wgsProjection = wgsProjection;
        _utmProjection = utmProjection;
        _mgrsProjection = mgrsProjection;
        _lamProjection = lamProjection;
        _w3wProjection = w3wProjection;
        _gkProjection = gkProjection;
        _bngProjection = bngProjection;
      });
    });
  }

  Future<Projection?> convertProjection(Projection projection, ProjectionType type) async {
    final completer = Completer<Projection?>();

    ProjectionService.convert(
        from: projection,
        toType: type,
        onComplete: (err, convertedProjection) {
          if (err != GemError.success) {
            completer.complete(null);
          } else {
            completer.complete(convertedProjection);
          }
        });

    return await completer.future;
  }
}
```

### Projections Panel

The `ProjectionsPanel` widget displays the coordinates in different projection systems. It is shown when a point is selected on the map.
```dart
class ProjectionsPanel extends StatelessWidget {
  final WGS84Projection? wgsProjection;
  final MGRSProjection? mgrsProjection;
  final UTMProjection? utmProjection;
  final LAMProjection? lamProjection;
  final W3WProjection? w3wProjection;
  final GKProjection? gkProjection;
  final BNGProjection? bngProjection;
  final VoidCallback onClose;
  const ProjectionsPanel(
      {super.key,
      this.wgsProjection,
      this.mgrsProjection,
      this.utmProjection,
      this.lamProjection,
      this.w3wProjection,
      this.gkProjection,
      this.bngProjection,
      required this.onClose});

  @override
  Widget build(BuildContext context) {
    return Stack(
      children: [
        Container(
          color: Colors.white,
          width: MediaQuery.of(context).size.width,
          child: Padding(
            padding: const EdgeInsets.only(bottom: 50.0, left: 20.0, right: 20.0, top: 10),
            child: Column(
              mainAxisSize: MainAxisSize.min,
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(
                  'WGS84: ${wgsProjection!.coordinates!.latitude.toStringAsFixed(6)},
                   ${wgsProjection!.coordinates!.longitude.toStringAsFixed(6)}',
                  style: const TextStyle(color: Colors.black, fontSize: 16),
                ),
                (bngProjection != null)
                    ? Text(
                        'BNG: ${bngProjection!.easting.toStringAsFixed(4)},
                         ${bngProjection!.northing.toStringAsFixed(4)}',
                        style: const TextStyle(color: Colors.black, fontSize: 16),
                      )
                    : const Text(
                        'BNG: Not available',
                        style: TextStyle(color: Colors.black, fontSize: 16),
                      ),
                (utmProjection != null)
                    ? Text(
                        'UTM: ${utmProjection!.x.toStringAsFixed(2)},
                         ${utmProjection!.y.toStringAsFixed(2)},
                          zone: ${utmProjection!.zone}, ${utmProjection!.hemisphere}',
                        style: const TextStyle(color: Colors.black, fontSize: 16),
                      )
                    : const Text(
                        'UTM: Not available',
                        style: TextStyle(color: Colors.black, fontSize: 16),
                      ),
                (mgrsProjection != null)
                    ? Text(
                        'MGRS: ${mgrsProjection!.zone}, ${mgrsProjection!.letters},
                         ${mgrsProjection!.easting.toStringAsFixed(2)},
                          ${mgrsProjection!.northing.toStringAsFixed(2)}',
                        style: const TextStyle(color: Colors.black, fontSize: 16),
                      )
                    : const Text(
                        'MGRS: Not available',
                        style: TextStyle(color: Colors.black, fontSize: 16),
                      ),
                (lamProjection != null)
                    ? Text(
                        'LAM: ${lamProjection!.x.toStringAsFixed(2)},
                         ${lamProjection!.y.toStringAsFixed(2)}',
                        style: const TextStyle(color: Colors.black, fontSize: 16),
                      )
                    : const Text(
                        'LAM: Not available',
                        style: TextStyle(color: Colors.black, fontSize: 16),
                      ),
                (w3wProjection != null)
                    ? Text(
                        'W3W: ${w3wProjection!.words}',
                        style: const TextStyle(color: Colors.black, fontSize: 16),
                      )
                    : const Text(
                        'W3W: Not available',
                        style: TextStyle(color: Colors.black, fontSize: 16),
                      ),
                (gkProjection != null)
                    ? Text(
                        'GK: ${gkProjection!.easting.toStringAsFixed(2)},
                         ${gkProjection!.northing.toStringAsFixed(2)},
                          zone: ${gkProjection!.zone}',
                        style: const TextStyle(color: Colors.black, fontSize: 16),
                      )
                    : const Text(
                        'GK: Not available',
                        style: TextStyle(color: Colors.black, fontSize: 16),
                      ),
              ],
            ),
          ),
        ),
        Positioned(
          top: 0,
          right: 0,
          child: IconButton(
            icon: const Icon(Icons.close, color: Colors.black),
            onPressed: onClose,
          ),
        ),
      ],
    );
  }
}
```


