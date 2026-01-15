---
description: Documentation for Route Bookmarks
title: Route Bookmarks
---

# Route Bookmarks

This example demonstrates how to create a Flutter app that allows users to save and manage route bookmarks using Maps SDK for Flutter.

## How it works

The example app demonstrates the following features:

- Display a map.

- Calculate and display routes on map.

- Save routes to history.

- Manage route history. 

### UI and Map Integration

This code sets up the basic structure of the app, including the map and the app bar. It also provides buttons in the app bar for building routes and accessing route bookmarks.
```dart
const projectApiToken = String.fromEnvironment("GEM_TOKEN");

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Route Bookmarks',
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
  late RouteBookmarks _routeBookmarks;
  TaskHandler? _routingHandler;
  bool _areRoutesBuilt = false;
  int _currentRouteIndex = 0;

  // Define 3 different route pairs
  final List<Map<String, Map<String, double>>> _routePairs = [
    {
      'departure': {'latitude': 48.85682, 'longitude': 2.34375}, // Paris
      'destination': {'latitude': 50.84644, 'longitude': 4.34587}, // Brussels
    },
    {
      'departure': {'latitude': 51.5074, 'longitude': -0.1278}, // London
      'destination': {'latitude': 52.5200, 'longitude': 13.4050}, // Berlin
    },
    {
      'departure': {'latitude': 41.9028, 'longitude': 12.4964}, // Rome
      'destination': {'latitude': 40.4168, 'longitude': -3.7038}, // Madrid
    },
  ];

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
        title: const Text('Route Bookmarks', style: TextStyle(color: Colors.white)),
        actions: [
          if (_routingHandler == null)
            IconButton(
              onPressed: _onHistoryButtonPressed,
              icon: const Icon(Icons.history, color: Colors.white),
              tooltip: 'Route History',
            ),
          if (!_areRoutesBuilt && _routingHandler == null)
            IconButton(
              onPressed: () => _onBuildRouteButtonPressed(context),
              icon: const Icon(Icons.route, color: Colors.white),
            ),
          if (_routingHandler != null)
            IconButton(
              onPressed: _onCancelRouteButtonPressed,
              icon: const Icon(Icons.stop, color: Colors.white),
            ),
          if (_areRoutesBuilt)
            IconButton(
              onPressed: _onClearRoutesButtonPressed,
              icon: const Icon(Icons.clear, color: Colors.white),
            ),
        ],
      ),
      body: GemMap(appAuthorization: projectApiToken, onMapCreated: _onMapCreated),
    );
  }

  void _onMapCreated(GemMapController mapController) {
    _mapController = mapController;
    _routeBookmarks = RouteBookmarks.create('route_history');
  }

  void _onBuildRouteButtonPressed(BuildContext context) {
    // Get current route pair
    final currentPair = _routePairs[_currentRouteIndex];
    final departure = currentPair['departure']!;
    final destination = currentPair['destination']!;

    final departureLandmark = Landmark.withLatLng(
      latitude: departure['latitude']!,
      longitude: departure['longitude']!,
    );
    final destinationLandmark = Landmark.withLatLng(
      latitude: destination['latitude']!,
      longitude: destination['longitude']!,
    );
    final routePreferences = RoutePreferences();

    _showSnackBar(
      context,
      message: 'Calculating route ${_currentRouteIndex + 1} of ${_routePairs.length}...',
    );

    _routingHandler = RoutingService.calculateRoute(
      [departureLandmark, destinationLandmark],
      routePreferences,
      (err, routes) {
        _routingHandler = null;
        ScaffoldMessenger.of(context).clearSnackBars();

        if (err == GemError.success) {
          final routesMap = _mapController.preferences.routes;

          for (final route in routes) {
            routesMap.add(route, route == routes.first);
          }

          _mapController.centerOnRoutes(routes: routes);

          // Save route to bookmarks
          _saveRouteToBookmarks(departureLandmark, destinationLandmark, routePreferences);

          setState(() {
            _areRoutesBuilt = true;
            // Cycle to next route pair for next time
            _currentRouteIndex = (_currentRouteIndex + 1) % _routePairs.length;
          });
        }
      },
    );

    setState(() {});
  }

  void _onClearRoutesButtonPressed() {
    _mapController.preferences.routes.clear();

    setState(() {
      _areRoutesBuilt = false;
      _routingHandler = null;
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

  void _onHistoryButtonPressed() async {
    // Clear existing routes before loading from history
    _mapController.preferences.routes.clear();

    // Reset page state
    setState(() {
      _areRoutesBuilt = false;
      _routingHandler = null;
    });

    final result = await Navigator.of(context).push(
      MaterialPageRoute(builder: (context) => RouteHistoryPage(routeBookmarks: _routeBookmarks)),
    );

    if (result != null && result is Map<String, dynamic>) {
      final waypoints = result['waypoints'] as List<Landmark>?;
      final preferences = result['preferences'] as RoutePreferences?;

      if (waypoints != null && waypoints.length >= 2 && preferences != null) {
        _calculateRouteFromHistory(waypoints, preferences);
      }
    }
  }

  void _calculateRouteFromHistory(List<Landmark> waypoints, RoutePreferences preferences) {
    _showSnackBar(context, message: 'Calculating route from history...');

    _routingHandler = RoutingService.calculateRoute(waypoints, preferences, (err, routes) {
      _routingHandler = null;
      ScaffoldMessenger.of(context).clearSnackBars();

      if (err == GemError.success) {
        final routesMap = _mapController.preferences.routes;

        for (final route in routes) {
          routesMap.add(route, route == routes.first);
        }

        _mapController.centerOnRoutes(routes: routes);

        setState(() {
          _areRoutesBuilt = true;
        });
      } else {
        _showSnackBar(
          context,
          message: 'Failed to calculate route',
          duration: const Duration(seconds: 3),
        );
      }
    });

    setState(() {});
  }

  void _saveRouteToBookmarks(
    Landmark departure,
    Landmark destination,
    RoutePreferences preferences,
  ) {
    final timestamp = DateTime.now();
    final routeName =
        'Route ${timestamp.day}/${timestamp.month} ${timestamp.hour}:
        ${timestamp.minute.toString().padLeft(2, '0')}
        :${timestamp.second.toString().padLeft(2, '0')}';

    _routeBookmarks.add(
      routeName,
      [departure, destination],
      preferences: preferences,
      overwrite: false,
    );
  }

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

### Route History Page

This code defines a separate page for viewing and managing saved route bookmarks. Users can select a route from history to recalculate and display it on the map.
```dart
class RouteHistoryPage extends StatefulWidget {
  final RouteBookmarks routeBookmarks;

  const RouteHistoryPage({super.key, required this.routeBookmarks});

  @override
  State<RouteHistoryPage> createState() => _RouteHistoryPageState();
}

class _RouteHistoryPageState extends State<RouteHistoryPage> {
  @override
  Widget build(BuildContext context) {
    final routeCount = widget.routeBookmarks.size;

    return Scaffold(
      appBar: AppBar(
        backgroundColor: Colors.deepPurple[900],
        foregroundColor: Colors.white,
        title: const Text('Route History', style: TextStyle(color: Colors.white)),
        actions: [
          if (routeCount > 0)
            IconButton(
              onPressed: _onClearAllPressed,
              icon: const Icon(Icons.delete_forever, color: Colors.white),
              tooltip: 'Clear all history',
            ),
        ],
      ),
      body: routeCount == 0
          ? const Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Icon(Icons.route, size: 64, color: Colors.grey),
                  SizedBox(height: 16),
                  Text('No routes in history', style: TextStyle(fontSize: 18, color: Colors.grey)),
                  SizedBox(height: 8),
                  Text(
                    'Calculate some routes to see them here',
                    style: TextStyle(fontSize: 14, color: Colors.grey),
                  ),
                ],
              ),
            )
          : ListView.builder(
              itemCount: routeCount,
              itemBuilder: (context, index) {
                final name = widget.routeBookmarks.getName(index);
                final waypoints = widget.routeBookmarks.getWaypoints(index);
                final timestamp = widget.routeBookmarks.getTimestamp(index);

                return Card(
                  margin: const EdgeInsets.symmetric(horizontal: 8, vertical: 4),
                  child: ListTile(
                    onTap: () => _onRouteTapped(index, waypoints),
                    leading: CircleAvatar(
                      backgroundColor: Colors.deepPurple[900],
                      child: Text('${index + 1}', style: const TextStyle(color: Colors.white)),
                    ),
                    title: Text(
                      name ?? 'Route ${index + 1}',
                      style: const TextStyle(fontWeight: FontWeight.bold),
                    ),
                    subtitle: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        if (waypoints != null && waypoints.length >= 2) ...[
                          const SizedBox(height: 4),
                          Text(
                            'From: ${_formatCoordinates(waypoints.first.coordinates)}',
                            style: const TextStyle(fontSize: 12),
                          ),
                          Text(
                            'To: ${_formatCoordinates(waypoints.last.coordinates)}',
                            style: const TextStyle(fontSize: 12),
                          ),
                        ],
                        if (timestamp != null) ...[
                          const SizedBox(height: 4),
                          Text(
                            'Saved: ${_formatDate(timestamp)}',
                            style: const TextStyle(fontSize: 11, color: Colors.grey),
                          ),
                        ],
                      ],
                    ),
                    trailing: IconButton(
                      icon: const Icon(Icons.delete, color: Colors.red),
                      onPressed: () => _onDeletePressed(index, name),
                    ),
                  ),
                );
              },
            ),
    );
  }

  void _onRouteTapped(int index, List<Landmark>? waypoints) {
    if (waypoints != null && waypoints.length >= 2) {
      // Pop with the selected route data
      Navigator.of(context).pop({'waypoints': waypoints, 'preferences': RoutePreferences()});
    }
  }

  String _formatCoordinates(Coordinates coords) {
    return '${coords.latitude.toStringAsFixed(4)}, ${coords.longitude.toStringAsFixed(4)}';
  }

  String _formatDate(DateTime date) {
    return '${date.day}/${date.month}/${date.year} ${date.hour}:
            ${date.minute.toString().padLeft(2, '0')}';
  }

  void _onDeletePressed(int index, String? name) {
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('Delete Route'),
        content: Text('Are you sure you want to delete "${name ?? 'Route ${index + 1}'}"?'),
        actions: [
          TextButton(onPressed: () => Navigator.of(context).pop(), child: const Text('Cancel')),
          TextButton(
            onPressed: () {
              widget.routeBookmarks.remove(index);
              Navigator.of(context).pop();
              setState(() {});
              ScaffoldMessenger.of(
                context,
              ).showSnackBar(const SnackBar(content: Text('Route deleted')));
            },
            child: const Text('Delete', style: TextStyle(color: Colors.red)),
          ),
        ],
      ),
    );
  }

  void _onClearAllPressed() {
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('Clear All History'),
        content: const Text('Are you sure you want to delete all routes from history?'),
        actions: [
          TextButton(onPressed: () => Navigator.of(context).pop(), child: const Text('Cancel')),
          TextButton(
            onPressed: () {
              widget.routeBookmarks.clear();
              Navigator.of(context).pop();
              setState(() {});
              ScaffoldMessenger.of(
                context,
              ).showSnackBar(const SnackBar(content: Text('All routes deleted')));
            },
            child: const Text('Clear All', style: TextStyle(color: Colors.red)),
          ),
        ],
      ),
    );
  }
}
```


