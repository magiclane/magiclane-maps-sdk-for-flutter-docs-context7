---
description: Documentation for Get Started Routing
title: Get Started Routing
---

# Get started with routing

Here’s a quick overview of what you can do with routing:

- Calculate routes from a start point to a destination.

- Include intermediary waypoints for multi-stop routes.

- Compute range routes to determine areas reachable within a specific range.

- Plan routes over predefined tracks.

- Customize routes with preferences like route types, restrictions, and more.

- Retrieve maneuvers and turn-by-turn instructions.

- Access detailed route profiles for further analysis.

## Calculate routes

You can calculate a route with the code below. This route is navigable, which means that later it is possible to do a navigation/ simulation on it.
```dart
// Define the departure.
final departureLandmark =
    Landmark.withLatLng(latitude: 48.85682, longitude: 2.34375);

// Define the destination.
final destinationLandmark =
    Landmark.withLatLng(latitude: 50.84644, longitude: 4.34587);

// Define the route preferences (all default).
final routePreferences = RoutePreferences();

TaskHandler? taskHandler = RoutingService.calculateRoute(
    [departureLandmark, destinationLandmark], routePreferences,
    (err, routes) {
        if (err == GemError.success) {
            showSnackbar("Number of routes: ${routes.length}");
        } else if (err == GemError.cancel) {
            showSnackbar("Route computation canceled");
        } else {
            showSnackbar("Error: $err");
        }
    });
```

The `RoutingService.calculateRoute` method returns `null` only when the computation fails to initiate. In such cases, calling `RouteService.cancelRoute(taskHandler)` is not possible. Error details will be delivered through the `onComplete` function of the `RoutingService.calculateRoute` method.

The `err` provided by the callback function can have the following values:
<table>
<tr>
<th>Value</th>
<th>Significance</th>
</tr>
<tr>
<td>`GemError.success`</td>
<td>successfully completed</td>
</tr>
<tr>
<td>`GemError.cancel`</td>
<td>cancelled by the user</td>
</tr>
<tr>
<td>`GemError.waypointAccess`</td>
<td>couldn't be found with the current preferences</td>
</tr>
<tr>
<td>`GemError.connectionRequired`</td>
<td>if allowOnlineCalculation = false in the routing preferences and the calculation can't be done on the client side due to missing data</td>
</tr>
<tr>
<td>`GemError.expired`</td>
<td>calculation can't be done on client side due to missing necessary data and the client world map data version is no longer supported by the online routing service</td>
</tr>
<tr>
<td>`GemError.routeTooLong`</td>
<td>routing was executed on the online service and the operation took too much time to complete (usually more than 1 min, depending on the server overload state)</td>
</tr>
<tr>
<td>`GemError.invalidated`</td>
<td>the offline map data changed ( offline map downloaded, erased, updated ) during the calculation</td>
</tr>
<tr>
<td>`GemError.noMemory`</td>
<td>routing engine couldn't allocate the necessary memory for the calculation</td>
</tr>
</table>

The previous example shows how to define your start and end points, set route preferences, and handle the callback for results. If needed, you can cancel the ongoing computation:
```dart
RoutingService.cancelRoute(taskHandler);
```

When the route is canceled, the callback will return `err` = `GemError.cancel`.

## Get ETA and traffic information

Once the route is computed, you can retrieve additional details like the estimated time of arrival (ETA) and traffic information. Here’s how you can access these:
```dart
TimeDistance td = route.getTimeDistance(activePart: false);

final totalDistance = td.totalDistanceM;
// same with:
//final totalDistance = td.unrestrictedDistanceM + td.restrictedDistanceM;

final totalDuration = td.totalTimeS;
// same with:
//final totalDuration = td.unrestrictedTimeS + td.restrictedTimeS;

// by default activePart = true
TimeDistance remainTd = route.getTimeDistance(activePart: true);

final totalRemainDistance = remainTd.totalDistanceM;
final totalRemainDuration = remainTd.totalTimeS;
```

By using the method `Route.getTimeDistance` we can get the time and distance for a route. If the `activePart` parameter is `false`, it means the distance is computed for the entire route initially computed, otherwise it is computed only for the active part (the part still remaining to be navigated). The default value for this parameter is `true`.

In the example `unrestricted` means the part of the route that is on public property and `restricted` the part of the route that is on private property. Time is measured in seconds and distance in meters.

If you want to gather traffic details, you can do so like this:
```dart
List<RouteTrafficEvent> trafficEvents = route.trafficEvents;

for (final event in trafficEvents) {
    RouteTransportMode transportMode = event.affectedTransportModes;
    String description = event.description;
    TrafficEventClass eventClass = event.eventClass;
    TrafficEventSeverity eventSeverity = event.eventSeverity;
    Coordinates from = event.from;
    Coordinates to = event.to;
    bool isRoadBlock = event.isRoadblock;
}
```

Check the [Traffic Events guide](../core/traffic-events) for more details.

## Display routes on map

After calculating the routes, they are not automatically displayed on the map. To visualize and center the map on the route, refer to the [display routes on maps](/guides/maps/display-map-items/display-routes) related documentation. The Maps SDK for Flutter offers extensive customization options, allowing for flexible preferences to tailor the display to your needs.

## Get the Terrain Profile

When computing the route we can choose to also build the `TerrainProfile` for the route.

In order to do that `RoutePreferences` must specify we want to also generate the `BuildTerrainProfile`:
```dart
final routePreferences = RoutePreferences(
  //highlight-start
  buildTerrainProfile: const BuildTerrainProfile(enable: true),
  //highlight-end
);
```

Setting the ``BuildTerrainProfile`` with the ``enable`` flag set to true to the preferences used within ``calculateRoute`` is mandatory for getting route terrain profile data.

Later, use the profile for elevation data or other terrain-related details:
```dart
RouteTerrainProfile? terrainProfile = route.terrainProfile;

if (terrainProfile != null) {
  double minElevation = terrainProfile.minElevation;
  double maxElevation = terrainProfile.maxElevation;
  int minElevDist = terrainProfile.minElevationDistance;
  int maxElevDist = terrainProfile.maxElevationDistance;
  double totalUp = terrainProfile.totalUp;
  double totalDown = terrainProfile.totalDown;

  // elevation at 100m from the route start
  double elevation = terrainProfile.getElevation(100);

  for (final section in terrainProfile.roadTypeSections) {
    RoadType roadType = section.type;
    int startDistance = section.startDistanceM;
  }

  for (final section in terrainProfile.surfaceSections) {
    SurfaceType surfaceType = section.type;
    int startDistance = section.startDistanceM;
  }

  for (final section in terrainProfile.climbSections) {
    Grade grade = section.grade;
    double slope = section.slope;
    int startDistanceM = section.startDistanceM;
    int endDistanceM = section.endDistanceM;
  }

  List<double> categs = [-16, -10, -7, -4, -1, 1, 4, 7, 10, 16];

  List<SteepSection> steepSections = terrainProfile.getSteepSections(categs);
  for (final section in steepSections) {
    int categ = section.categ;
    int startDistanceM = section.startDistanceM;
  }
}
```

`RoadType` possible values are: motorways, stateRoad, road, street, cycleway, path, singleTrack.

`SurfaceType` possible values are: asphalt, paved, unpaved, unknown.

For more information see the [Route Profile example](/examples/routing-navigation/route-profile).

## Get the route segments and instructions

Once a route has been successfully computed, you can retrieve a detailed list of its `segments`. Each segment represents the portion of the route between two consecutive waypoints and includes its own set of `route instructions`.

For instance, if a route is computed with five waypoints, it will consist of four segments, each with distinct instructions.

In the case of public transit routes, segments can represent either pedestrian paths or public transit sections.

Here’s an example of how to access and use this information, focusing on some key `RouteInstruction` properties:

<table>
  <tr>
    <th>Field</th>
    <th>Type</th>
    <th>Explanation</th>
  </tr>
  <tr>
    <td>traveledTimeDistance</td>
    <td>TimeDistance</td>
    <td>Time and distance from the beginning of the route.</td>
  </tr>
  <tr>
    <td>remainingTravelTimeDistance</td>
    <td>TimeDistance</td>
    <td>Time and distance to the end of the route.</td>
  </tr>
    <tr>
    <td>coordinates</td>
    <td>Coordinates</td>
    <td>The coordinates indicating the location of the instruction.</td>
  </tr>
  <tr>
    <td>remainTravelTimeDistToNextWaypoint</td>
    <td>TimeDistance</td>
    <td>Time and distance until the next waypoint.</td>
  </tr>
  <tr>
    <td>timeDistanceToNextTurn</td>
    <td>TimeDistance</td>
    <td>Time and distance until the next instruction.</td>
  </tr>
  <tr>
    <td>turnDetails</td>
    <td>TurnDetails</td>
    <td>Get full details for the turn.</td>
  </tr>
  <tr>
    <td>turnInstruction</td>
    <td>String</td>
    <td>Get textual description for the turn.</td>
  </tr>
  <tr>
    <td>roadInfo</td>
    <td>List&lt;RoadInfo&gt;</td>
    <td>Get road information.</td>
  </tr>
  <tr>
    <td>hasFollowRoadInfo</td>
    <td>bool</td>
    <td>Check is the road has follow road information.</td>
  </tr>
  <tr>
    <td>followRoadInstruction</td>
    <td>String</td>
    <td>Get textual description for the follow road information.</td>
  </tr>
  <tr>
    <td>countryCodeISO</td>
    <td>String</td>
    <td>Get ISO 3166-1 alpha-3 country code for the navigation instruction.</td>
  </tr>
    <tr>
    <td>exitDetails</td>
    <td>String</td>
    <td>Get the exit route instruction text.</td>
  </tr>
    <tr>
    <td>signpostInstruction</td>
    <td>String</td>
    <td>Get textual description for the signpost information.</td>
  </tr>
    <tr>
    <td>signpostDetails</td>
    <td>SignpostDetails</td>
    <td>Get extended signpost details.</td>
  </tr>
    <tr>
    <td>roadInfoImg</td>
    <td>RoadInfoImg</td>
    <td>Get customizable road image. The user is responsible to check if the image is valid.</td>
  </tr>
    <tr>
    <td>turnImg</td>
    <td>Img</td>
    <td>Get turn image. The user is responsible to check if the image is valid.</td>
  </tr>
    <tr>
    <td>realisticNextTurnImg</td>
    <td>AbstractGeometryImg</td>
    <td>Get customizable image for the realistic turn information. The user is resposible to check if the image is valid.</td>
  </tr>
</table>

Data from the instruction list above is obtained via the following methods of `RouteInstruction`:

-  `turnInstruction` : Bear left onto A 5.

-  `followRoadInstruction` : Follow A 5 for 132m.

-  `traveledTimeDistance.totalDistanceM` : 6.2km. (after formatting to km)

-  `turnDetails.abstractGeometryImg.getRenderableImageBytes(renderSettings: AbstractGeometryImageRenderSettings(),size: Size(100, 100))` : Instruction image or null when image is invalid.

## Relevant examples demonstrating routing related features

- [Calculate Route](/examples/routing-navigation/calculate-route)

- [Route Profile](/examples/routing-navigation/route-profile)

- [Route Instructions](/examples/routing-navigation/route-instructions)

- [Finger Route](/examples/routing-navigation/finger-route)

- [Range Finder](/examples/routing-navigation/range-finder)

- [GPX Route](/examples/routing-navigation/gpx-route)

- [Truck Profile](/examples/routing-navigation/truck-profile)

- [Public Transit](/examples/routing-navigation/public-transit)

- [Offline Routing](/examples/routing-navigation/offline-routing)

- [Multi Map Routing](/examples/routing-navigation/multimap-routing)

- [GPX Thumbnail Image](/examples/routing-navigation/gpx-thumbnail-image)
