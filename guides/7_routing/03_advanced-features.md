---
description: Documentation for Advanced Features
title: Advanced Features
---

# Advanced features

## Compute route ranges

In order to compute a route range we need to:

- Specify in the `RoutePreferences` the most important route preferences (others can also be used):

     - `routeRanges` list containing a list of range values, one for each route we compute. Measurement units are corresponding to the specified `routeType` (see the table below)

     - [optional] `transportMode` (by default `TransportMode.car`)

     - [optional] `routeType` (can be `fastest`, `economic`, `shortest` - by default is fastest)

     - [optional] `routeRangesQuality` ( a value in the interval [0, 100], default 100) representing the quality of the generated polygons.

- The list of landmarks will contain only one landmark, the starting point for the route range computation.

| Preference                          | Measurement unit     |
|-------------------------------------|----------------------|
| fastest                             | seconds              |
| shortest                            | meters               |
| economic                            | Wh                   |

Routes computed using route ranges are **not navigable**.

The `RouteType.scenic` route type is not supported for route ranges.

Route can be computed with a code like the following. It is a range route computation because it only has a simple `Landmark` and `routeRanges` contains values (in this case 2 routes will be computed).
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

The computed routes can be displayed on the map, just like any regular route, with the only difference that the additional settings `RouteRenderSettings.m_fillColor` is used to define the polygon fill color.

## Compute path based routes

A `Path` is a structure containing a list of coordinates (a track). It can be created based on:

- custom coordinates specified by the user

- coordinates recorded in a GPX file

- coordinates obtained by doing a finger draw on the map

A **Path backed landmark** is a special kind of `Landmark` that has a `Path` inside it.

Sometimes we want to compute routes based on a list of one or more **Path backed landmark**(s) and optionally some regular `Landmark`(s). In this case the result will only contain one route. The path provided as waypoint track is used as a hint for the routing algorithm.

You can see an example below (the highlighted area represents the code necessary to create the list with one element of type landmark built based on a path): 
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
List<Landmark> landmarkList = gemPath.toLandmarkList();
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

The `Path` object associated to a path based landmark can be modified using the `trackData` setter available on the `Landmark` object.
See the [Landmarks guide](../core/landmarks) for more details about this.

When computing a route based on a path backed landmark **and** non-path backed landmarks, it is mandatory to set the `accurateTrackMatch` field from `RoutePreferences` to `true`. Otherwise, the routing computation will fail with a `GemError.unsupported` error.

The `isTrackResume` field from `RoutePreferences` can also be set to configure the behaviour of the routing engine when one track based landmark is used as a waypoint toghether with other landmarks.
If this field is set to `true`, the routing engine will try to match the entire track of the path based landmark.
Otherwise, if set to `false`, only the end point of the track will be used as waypoints.

## Computing a route based on a GPX file

You can compute a route based on a GPX file by using the `path based landmark` described in the previous section. The only difference is how we compute the `gemPath`.
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
final landmarkList = gemPath.toLandmarkList();

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

## Finger drawn path

When necessary, it is possible to record a path based on drawing with the finger on the map. 

It is also possible to record multiple paths. In this situation a straight line is added between any 2 consecutive finger drawn paths.

When you want to enter this recording mode:
```dart
mapController.enableDrawMarkersMode();
```

When you want to exit this mode, you can get the generated `List<Landmark>` with the following:
```dart
List<Landmark> landmarks = mapController.disableDrawMarkersMode();

final routePreferences = RoutePreferences(
    accurateTrackMatch: false, ignoreRestrictionsOverTrack: true);

TaskHandler? taskHandler = RoutingService.calculateRoute(
    landmarks, routePreferences, (err, routes) {
        // handle result
    });
```

The resulted `List<Landmark>` will only contain one element, a path based `Landmark`.

## Compute public transit routes

In order to compute a public transit route we need to set the `transportMode` field in the `RoutePreferences` like this:
```dart
// Define the route preferences with public transport mode.
final routePreferences =
    RoutePreferences(transportMode: RouteTransportMode.public);
```

Public transit routes are not navigable.

The full source code to compute a public transit route and handle it could look like this:
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

Once routes are computed, if the computation was for public transport route, you can convert a resulted route to a public transit route via `toPtRoute()`. After that you have full access to the methods specific to this kind of route.

A public transit route is a sequence of one or more segments. Each segment is either a walking segment, either a public transit segment. You can determine the segment type based on the `TransitType`.

`TransitType` can have the following values: walk, bus, underground, railway, tram, waterTransport, other, sharedBike, sharedScooter, sharedCar, unknown.

Other settings related to public transit (such as departure/arrival time) can be specified within the `RoutePreferences` object passed to the `calculateRoute` method:
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

## Export a Route to file

The `exportToFile` method allows you to export a route from `RouteBookmarks` into a file on disk. This makes it possible to store the route for later use or share it with other systems.

The file will be saved at the exact location provided in the **filePath** parameter, so always ensure the directory exists and is writable.
```dart
final error = routeBookmark.exportToFile(index, filePath);
```

:::tip[TIP]

When exporting a route, make sure to handle possible errors:  

- **`GemError.notFound`** — This occurs if the given route index does not exist.

- **`GemError.io`** — This occurs if the file cannot be created or written to. 

## Export a Route as String

The `exportAs` method allows you to export a route into a textual representation. The returned value is a `String` containing the full route data in the requested format.  
This makes it easy to store the route as a file or share it with other applications that support formats like GPX, KML, NMEA, or GeoJSON.
```dart
final dataGpx = routes.first.exportAs(PathFileFormat.gpx);
// You now have the full GPX as a string
```

## Relevant examples demonstrating routing related features

- [Finger Route](/examples/routing-navigation/finger-route)

- [GPX Thumbnail Image](/examples/routing-navigation/gpx-thumbnail-image)

- [GPX Routing Thumbnail Image](/examples/routing-navigation/gpx-routing-thumbnail-image)

- [Range Finder](/examples/routing-navigation/range-finder)
