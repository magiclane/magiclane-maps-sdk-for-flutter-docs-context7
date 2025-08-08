---
description: Documentation for Range Finder
title: Range Finder
---

# Range Finder

In this guide, you will learn how to implement route range calculation from a Point of Interest (POI) using the Maps SDK for Flutter. This example demonstrates how to display a map, tap on a landmark, and calculate route ranges based on different transport modes and preferences.

## How it Works

This example demonstrates the following features:

- Allow users to interact with a map by tapping landmarks to focus on specific Points of Interest (POIs).

- Perform route range calculations from selected POIs using preferences such as transport mode.

### UI and Map Integration

The following code creates a user interface featuring a `GemMap` widget and an app bar. When a Point of Interest (POI) is selected on the map, a range finder panel appears at the bottom of the screen.

Define the main application widget, MyApp.
```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Range Finder',
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

Within _MyHomePageState , define the necessary state variables and methods to interact with the map and manage routes.
```dart
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
        toolbarHeight: 50,
        backgroundColor: Colors.deepPurple[900],
        title: const Text('Range Finder', style: TextStyle(color: Colors.white)),
      ),
      body: Stack(
        children: [
          GemMap(
            key: ValueKey("GemMap"),
            onMapCreated: _onMapCreated,
            appAuthorization: projectApiToken,
          ),
          if (_focusedLandmark != null)
            Align(
                alignment: Alignment.bottomCenter,
                child: RangesPanel(
                  onCancelTap: _onCancelLandmarkPanelTap,
                  landmark: _focusedLandmark!,
                  mapController: _mapController,
              ),
            ),
        ],
      ),
    );
  }

  void _onMapCreated(GemMapController controller) {
    _mapController = controller;
    _registerLandmarkTapCallback();
  }

  void _registerLandmarkTapCallback() {
    _mapController.registerTouchCallback((pos) async {
      _mapController.setCursorScreenPosition(pos);
      final landmarks = _mapController.cursorSelectionLandmarks();

      if (landmarks.isEmpty) {
        return;
      }

      _mapController.activateHighlight(landmarks);
      final lmk = landmarks[0];
      setState(() {
        _focusedLandmark = lmk;
      });

      _mapController.centerOnCoordinates(lmk.coordinates);
    });
  }

  void _onCancelLandmarkPanelTap() {
    _mapController.deactivateAllHighlights();
    _mapController.preferences.routes.clear();
    setState(() {
      _focusedLandmark = null;
    });
  }
}
```

### Range Calculation and Preferences

The RangesPanel widget handles the UI and logic for calculating and displaying route ranges. Here are the critical parts:

### Define State Variables

Define state variables to hold user preferences and the calculated route ranges.
```dart
class _RangesPanelState extends State<RangesPanel> {
  int _rangeValue = 3600;
  RouteTransportMode _transportMode = RouteTransportMode.car;
  RouteType _routeType = RouteType.fastest;
  bool _avoidMotorways = false;
  bool _avoidTollRoads = false;
  bool _avoidFerries = false;
  bool _avoidUnpavedRoads = false;
  BikeProfile _bikeProfile = BikeProfile.city;
  double _hillsValue = 0;
  TrafficAvoidance _trafficAvoidance = TrafficAvoidance.roadblocks;
  List<Range> routeRanges = [];
}
```

### Calculate Route Ranges

Use the RoutingService to calculate route ranges based on user preferences.
```dart
void _onAddRouteRangeButtonPressed(BuildContext context) {
  if (!_doesRouteRangeExist()) {
    _showSnackBar(context, message: "The route is being calculated.");

    RoutingService.calculateRoute([widget.landmark], _getRoutePreferences(),
        (err, routes) {
      ScaffoldMessenger.of(context).clearSnackBars();

      if (err == GemError.success) {
        final routesMap = widget.mapController.preferences.routes;
        final randomColor = Color.fromARGB(128, Random().nextInt(200),
            Random().nextInt(200), Random().nextInt(200));
        RouteRenderSettings settings =
            RouteRenderSettings(fillColor: randomColor);
        routesMap.add(routes!.first, true, routeRenderSettings: settings);
        _centerOnRouteRange(routes.first);

        setState(() {
          _addNewRouteRange(routes.first, randomColor);
        });
      }
    });

    setState(() {});
  }
}
```

### Define Route Preferences

Create a method to build route preferences based on user inputs.
```dart
RoutePreferences _getRoutePreferences() {
  switch (_transportMode) {
    case RouteTransportMode.car:
      return RoutePreferences(
        avoidMotorways: _avoidMotorways,
        avoidTollRoads: _avoidTollRoads,
        avoidFerries: _avoidFerries,
        avoidUnpavedRoads: _avoidUnpavedRoads,
        transportMode: _transportMode,
        routeType: _routeType,
        routeRanges: [_rangeValue],
      );
    case RouteTransportMode.lorry:
      return RoutePreferences(
        avoidMotorways: _avoidMotorways,
        avoidTollRoads: _avoidTollRoads,
        avoidFerries: _avoidFerries,
        avoidUnpavedRoads: _avoidUnpavedRoads,
        transportMode: _transportMode,
        routeType: _routeType,
        routeRanges: [_rangeValue],
        avoidTraffic: _trafficAvoidance,
      );
    case RouteTransportMode.pedestrian:
      return RoutePreferences(
        avoidFerries: _avoidFerries,
        avoidUnpavedRoads: _avoidUnpavedRoads,
        transportMode: _transportMode,
        routeRanges: [_rangeValue],
      );
    case RouteTransportMode.bicycle:
      return RoutePreferences(
        avoidFerries: _avoidFerries,
        avoidUnpavedRoads: _avoidUnpavedRoads,
        transportMode: _transportMode,
        routeType: _routeType,
        routeRanges: [_rangeValue],
        avoidBikingHillFactor: _hillsValue,
        bikeProfile: BikeProfileElectricBikeProfile(
            profile: _bikeProfile, eProfile: ElectricBikeProfile()),
      );
    default:
      return RoutePreferences();
  }
}
```

### Handle User Interactions

Methods to manage user interactions, such as deleting, toggling, and centering on route ranges.
```dart
void _deleteRouteRange(int index) {
  widget.mapController.preferences.routes.remove(routeRanges[index].route);
  setState(() {
    routeRanges.removeAt(index);
  });
}

void _toggleRouteRange(int index) {
  if (routeRanges[index].isEnabled) {
    widget.mapController.preferences.routes.remove(routeRanges[index].route);
    return;
  } else {
    RouteRenderSettings settings =
        RouteRenderSettings(fillColor: routeRanges[index].color);
    widget.mapController.preferences.routes
        .add(routeRanges[index].route, true, routeRenderSettings: settings);
    _centerOnRouteRange(routeRanges[index].route);
  }
}

String _getRouteRangeValueString() {
    final String valueString =
        (_routeType == RouteType.fastest)
            ? convertDuration(_rangeValue)
            : (_routeType == RouteType.economic)
            ? convertWh(_rangeValue)
            : convertDistance(_rangeValue);
    return valueString;
  }

  bool _doesRouteRangeExist() {
    bool exists = routeRanges.any(
      (range) =>
          range.transportMode == _transportMode &&
          range.value == _getRouteRangeValueString(),
    );
    return exists;
  }

  void _addNewRouteRange(Route route, Color color) {
    Range newRange = Range(
      route: route,
      color: color,
      transportMode: _transportMode,
      value: _getRouteRangeValueString(),
      isEnabled: true,
    );
    routeRanges.add(newRange);
  }

void _centerOnRouteRange(Route route) {
  const appbarHeight = 50;
  const padding = 20;

  widget.mapController.centerOnRoute(route,
      screenRect: RectType(
        x: 0,
        y: (appbarHeight + padding * MediaQuery.of(context).devicePixelRatio)
            .toInt(),
        width: (MediaQuery.of(context).size.width *
                MediaQuery.of(context).devicePixelRatio)
            .toInt(),
        height: ((MediaQuery.of(context).size.height / 2 -
                    appbarHeight -
                    2 * padding * MediaQuery.of(context).devicePixelRatio) *
                  MediaQuery.of(context).devicePixelRatio)
            .toInt(),
      ),
    );
  }

void _showSnackBar(BuildContext context,
    {required String message, Duration duration = const Duration(hours: 1)}) {
  final snackBar = SnackBar(
    content: Text(message),
    duration: duration,
  );

  ScaffoldMessenger.of(context).showSnackBar(snackBar);
}
```

You can start calculating a range by tapping the + button after adjusting your specifications for the routes.
