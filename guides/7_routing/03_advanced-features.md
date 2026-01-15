---
description: Documentation for Advanced Features
title: Advanced Features
---

# Advanced features

This guide covers advanced routing features including route ranges, path-based routes, and public transit routing.

---

## Compute route ranges

To compute a route range:

- Specify in the `RoutePreferences` the most important route preferences (others can also be used):

     - `routeRanges` list containing a list of range values, one for each route we compute. Measurement units are corresponding to the specified `routeType` (see the table below)

     - [optional] `transportMode` (by default `TransportMode.car`)

     - [optional] `routeType` (can be `fastest`, `economic`, `shortest` - by default is fastest)

     - [optional] `routeRangesQuality` (a value in the interval [0, 100], default 100) representing the quality of the generated polygons

- The list of landmarks will contain only one landmark, the starting point for the route range computation

**Measurement units by route type:**

- **fastest** - seconds

- **shortest** - meters

- **economic** - Wh

Routes computed using route ranges are **not navigable**.

The `RouteType.scenic` route type is not supported for route ranges.

Compute a range route using a single `Landmark` and multiple `routeRanges` values:
```dart
// Define the departure.
final startLandmark =
    Landmark.withLatLng(latitude: 48.85682, longitude: 2.34375);

// Define the route preferences.
// Compute 2 ranges, 30 min and 60 min
final routePreferences = RoutePreferences(
    routeType: RouteType.fastest,
    routeRanges: [1800, 3600],
);

final taskHandler = RoutingService.calculateRoute(
    [startLandmark], routePreferences, (err, routes) {
        if (err == GemError.success) {
            showSnackbar("Route range computed");
        } else if (err == GemError.cancel) {
            showSnackbar("Route computation canceled");
        } else {
            showSnackbar("Error: $err");
        }
    });
```

Display computed routes on the map like regular routes. Use `RouteRenderSettings.m_fillColor` to define the polygon fill color.

---

## Compute path-based routes

A `Path` contains a list of coordinates (a track) created from:

- Custom coordinates

- GPX file coordinates

- Finger-drawn map coordinates

A **path-backed landmark** is a `Landmark` containing a `Path`. Compute routes using one or more path-backed landmarks combined with optional regular landmarks. The path serves as a hint for the routing algorithm, and the result contains only one route.
```dart
//highlight-start
final coords = [
    Coordinates(latitude: 40.786, longitude: -74.202),
    Coordinates(latitude: 40.690, longitude: -74.209),
    Coordinates(latitude: 40.695, longitude: -73.814),
    Coordinates(latitude: 40.782, longitude: -73.710),
];

Path gemPath = Path.fromCoordinates(coords);

// A list containing only one Path backed Landmark
List<Landmark> landmarkList = gemPath.landmarkList;
//highlight-end

// Define the route preferences.
final routePreferences = RoutePreferences();

final taskHandler = RoutingService.calculateRoute(
    landmarkList, routePreferences, (err, routes) {
        if (err == GemError.success) {
            showSnackbar("Number of routes: ${routes!.length}");
        } else if (err == GemError.cancel) {
            showSnackbar("Route computation canceled");
        } else {
            showSnackbar("Error: $err");
        }
});
```

Modify the `Path` object using the `trackData` setter on the `Landmark` object. See the [Landmarks guide](../core/landmarks) for details.

When computing a route with both path-backed and non-path-backed landmarks, set `accurateTrackMatch` to `true` in `RoutePreferences`. Otherwise, routing computation fails with `GemError.unsupported`.

Configure the routing engine behavior with the `isTrackResume` field:

- **`true`** - Matches the entire track of the path-backed landmark

- **`false`** - Uses only the end point as a waypoint

---

## Compute routes from GPX files

Compute a route from a GPX file using a path-based landmark. The only difference is creating the `gemPath` from the file:
```dart
//highlight-start
File gpxFile = await provideFile("recorded_route.gpx");

//Return if GPX file is not exists
if (!await gpxFile.exists()) {
    return showSnackbar('GPX file does not exist (${gpxFile.path})');
}

final pathData = Uint8List.fromList(await gpxFile.readAsBytes());

//Get landmarklist containing all GPX points from file.
final gemPath = Path.create(data: pathData, format: PathFileFormat.gpx);
//highlight-end

// LandmarkList will contain only one path based landmark.
final landmarkList = gemPath.landmarkList;

// Define the route preferences.
final routePreferences =
    RoutePreferences(transportMode: RouteTransportMode.bicycle);

RoutingService.calculateRoute(
    landmarkList,
    routePreferences,
    (err, routes) {
        // handle result
    },
);
```

---

## Finger drawn paths

Record a path by drawing with your finger on the map. When recording multiple paths, straight lines connect consecutive drawn segments.

Enable drawing mode:
```dart
mapController.enableDrawMarkersMode();
```

Exit drawing mode and retrieve the generated landmarks:
```dart
List<Landmark> landmarks = mapController.disableDrawMarkersMode();

final routePreferences = RoutePreferences(
    accurateTrackMatch: false, ignoreRestrictionsOverTrack: true);

TaskHandler? taskHandler = RoutingService.calculateRoute(
    landmarks, routePreferences, (err, routes) {
        // handle result
    });
```

The resulting `List<Landmark>` contains one path-based `Landmark`.

---

## Compute public transit routes

Set the `transportMode` field in `RoutePreferences` to compute public transit routes:
```dart
// Define the route preferences with public transport mode.
final routePreferences =
    RoutePreferences(transportMode: RouteTransportMode.public);
```

Public transit routes are not navigable.

Compute and handle a public transit route:
```dart
// Define the departure.
final departureLandmark =
    Landmark.withLatLng(latitude: 45.6646, longitude: 25.5872);

// Define the destination.
final destinationLandmark =
    Landmark.withLatLng(latitude: 45.6578, longitude: 25.6233);

// Define the route preferences with public transport mode.
final routePreferences =
    RoutePreferences(transportMode: RouteTransportMode.public);

TaskHandler? taskHandler = RoutingService.calculateRoute(
    [departureLandmark, destinationLandmark], routePreferences,
    (err, routes) {
    if (err == GemError.success) {
        if (routes.isNotEmpty) {
            // Get the routes collection from map preferences.
            final routesMap = mapController.preferences.routes;

            // Display the routes on map.
            for (final route in routes) {
                routesMap.add(route, route == routes.first,
                    label: route == routes.first ? "Route" : null);
            }

            // Convert normal route to PTRoute
            final ptRoute = routes.first.toPTRoute();

            // Convert each segment to PTRouteSegment
            final ptSegments =
                ptRoute!.segments.map((seg) => seg.toPTRouteSegment()).toList();

            for(final segment in ptSegments) {
                TransitType transitType = segment.transitType;

                if(segment.isCommon) { // PT segment
                    List<PTRouteInstruction> ptInstructions = segment.instructions.map((e) => e.toPTRouteInstruction()).toList();
                    for(final ptInstr in ptInstructions) {
                        // handle public transit instruction
                        String stationName = ptInstr.name;
                        DateTime? departure = ptInstr.departureTime;
                        DateTime? arrival = ptInstr.arrivalTime;
                        // ...
                    }
                } else { // walk segment
                    List<RouteInstruction> instructions = segment.instructions;
                    for(final walkInstr in instructions) {
                        // handle walk instruction
                    }
                }
            }
        }
    }
});
```

Convert computed routes to public transit routes using `toPtRoute()` to access public transit-specific methods.

A public transit route contains one or more segments. Each segment is either a walking or public transit segment. Determine the segment type using `TransitType`.

**Available `TransitType` values:**
walk, bus, underground, railway, tram, waterTransport, other, sharedBike, sharedScooter, sharedCar, unknown

Specify departure/arrival time and other public transit settings in the `RoutePreferences` object:
```dart
final customRoutePreferences = RoutePreferences(
    transportMode: RouteTransportMode.public,
    // The arrival time is set to one hour from now.
    algorithmType: PTAlgorithmType.arrival,
    timestamp: DateTime.now().add(Duration(hours: 1)),
    // Sort the routes by the best time.
    sortingStrategy: PTSortingStrategy.bestTime,
    // Accessibility preferences
    useBikes: false,
    useWheelchair: false,
);
```

---

## Export routes to files

Export a route from `RouteBookmarks` to a file on disk for later use or sharing. Ensure the directory exists and is writable before saving.
```dart
final error = routeBookmark.exportToFile(index, filePath);
```

Handle possible errors when exporting:

- **`GemError.notFound`** - Route index does not exist

- **`GemError.io`** - File cannot be created or written

## Export routes as strings

Export a route to a textual format using `exportAs`. The method returns a `String` containing the full route data in formats like GPX, KML, NMEA, or GeoJSON.
```dart
final dataGpx = routes.first.exportAs(PathFileFormat.gpx);
// Full GPX data as a string
```

---

## Relevant examples demonstrating routing related features

- [Finger Route](/examples/routing-navigation/finger-route)

- [GPX Thumbnail Image](/examples/routing-navigation/gpx-thumbnail-image)

- [GPX Routing Thumbnail Image](/examples/routing-navigation/gpx-routing-thumbnail-image)

- [Range Finder](/examples/routing-navigation/range-finder)
