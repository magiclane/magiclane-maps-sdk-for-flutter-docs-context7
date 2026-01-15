---
description: Documentation for Routes
title: Routes
---

# Routes

A route represents a navigable path between two or more landmarks (waypoints), including distance, estimated time, and navigation instructions.

Compute routes in different ways:

- **Waypoint-based** - Based on 2 or more landmarks (navigable)

- **Over-track** - Based on a predefined `path` from GPX files or other sources (navigable)

- **Route ranges** - Not navigable, without segments or instructions

Navigable routes consist of segments. Each segment represents the portion between consecutive waypoints with its own route instructions.

---

## Create Routes

Routes cannot be instantiated directly. Compute them based on a list of landmarks. See [Get started with Routing](/guides/routing/get-started-routing) for details.

Calculating a route does not automatically display it on the map. See [Display routes](/guides/maps/display-map-items/display-routes) for instructions.

---

## Route Types

The SDK supports multiple route types, each tailored for specific use cases:

- **Normal routes** - Standard routes for typical navigation

- **Public transport (PT) routes** - Routes using public transport with frequency, ticket info, and transit-specific data

- **Over-track (OT) routes** - Routes based on predefined paths from GPX files or drawn routes

- **Electric vehicle (EV) routes** - Routes for EVs with charging station information (not fully implemented)

### Route Classes

Each route type has specific classes:

| **Route Type**            | **Route Class** | **Segment Class**      | **Instruction Class**     |
|---------------------------|-----------------|------------------------|---------------------------|
| Normal Route              | `Route`         | `RouteSegment`         | `RouteInstruction`        |
| Public Transport Route    | `PTRoute`       | `PTRouteSegment`       | `PTRouteInstruction`      |
| Over-Track Route          | `OTRoute`       | Not Available          | Not Available             |
| Electric Vehicle Route    | `EVRoute`       | `EVRouteSegment`       | `EVRouteInstruction`      |

These classes extend base classes (`RouteBase`, `RouteSegmentBase`, `RouteInstructionBase`) that provide common features.

---

## Route Structure

Key route characteristics:

<table>
  <tr>
    <th>Field</th>
    <th>Type</th>
    <th>Explanation</th>
  </tr>
  <tr>
    <td>geographicArea</td>
    <td>GeographicArea</td>
    <td>A geographic boundary or region covered by the route.</td>
  </tr>
  <tr>
    <td>polygonGeographicArea</td>
    <td>PolygonGeographicArea</td>
    <td>A polygon representing the geographic area as a series of connected points.</td>
  </tr>
  <tr>
    <td>tilesGeographicArea</td>
    <td>TilesCollectionGeographicArea</td>
    <td>A collection of map tiles representing the geographic area.</td>
  </tr>
  <tr>
    <td>dominantRoads</td>
    <td>List&lt;String&gt;</td>
    <td>A list of road names or identifiers that dominate the route.</td>
  </tr>
  <tr>
    <td>hasFerryConnections</td>
    <td>bool</td>
    <td>Indicates whether the route includes ferry connections.</td>
  </tr>
  <tr>
    <td>hasTollRoads</td>
    <td>bool</td>
    <td>Indicates whether the route includes toll roads.</td>
  </tr>
  <tr>
    <td>tollSections</td>
    <td>List of TollSection</td>
    <td>The list of toll sections along the route.</td>
  </tr>
  <tr>
    <td>incursCosts</td>
    <td>bool</td>
    <td>Specifies if the route incurs any monetary costs, such as tolls or fees [DEPRECATED - same as hasTollRoads]</td>
  </tr>
  <tr>
    <td>routeStatus</td>
    <td>RouteStatus</td>
    <td>If we are navigating a route and we deviate from it, the route is recalculated. This means that the `routeStatus` might signal that route is computed or we don't have internet connection, or even that on recomputation we got an error.</td>
  </tr>
  <tr>
    <td>terrainProfile</td>
    <td>List&lt;double&gt;</td>
    <td>A profile of terrain elevations along the route, represented as a list of elevation values.</td>
  </tr>
  <tr>
    <td>segments</td>
    <td>List&lt;RouteSegment&gt;</td>
    <td>
        <div>A collection of route segments, each representing a specific portion of the route.</div>
        <div>Segments are split based on the initial waypoints that were used to compute the route.</div>
        <div>For Public Transit routes a segment is either a pedestrian part or a public transit part.</div>
    </td>
  </tr>
  <tr>
    <td>trafficEvents</td>
    <td>List&lt;RouteTrafficEvent&gt;</td>
    <td>A list of traffic events, such as delays or road closures, affecting the route.</td>
  </tr>
</table>

---

## RouteSegment Structure

A route segment represents the portion between two consecutive waypoints. For public transport routes, segments are categorized as pedestrian or public transit sections.  

<table>
  <thead>
    <tr>
      <th>Field/Method</th>
      <th>Return Type</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>`waypoints`</td>
      <td>`List<Landmark>`</td>
      <td>Retrieves the list of landmarks representing the start and end waypoints of the route segment.</td>
    </tr>
    <tr>
      <td>`timeDistance`</td>
      <td>`TimeDistance`</td>
      <td>Provides the length in meters and estimated travel time in seconds for the route segment.</td>
    </tr>
    <tr>
      <td>`geographicArea`</td>
      <td>`RectangleGeographicArea`</td>
      <td>Retrieves the smallest rectangle enclosing the geographic area of the route.</td>
    </tr>
    <tr>
      <td>`incursCosts`</td>
      <td>`bool`</td>
      <td>Checks whether traveling the route or segment incurs a cost to the user.</td>
    </tr>
    <tr>
      <td>`summary`</td>
      <td>`String`</td>
      <td>Provides a summary of the route segment.</td>
    </tr>
    <tr>
      <td>`instructions`</td>
      <td>`List<RouteInstruction>`</td>
      <td>Retrieves the list of route instructions for the segment.</td>
    </tr>
    <tr>
      <td>`isCommon`</td>
      <td>`bool`</td>
      <td>Indicates whether the segment shares the same travel mode as the parent route. Mostly used within public transport routes.</td>
    </tr>
    <tr>
      <td>tollSections</td>
      <td>`List<TollSection>`</td>
    <td>The list of toll sections along the route segment.</td>
  </tr>
  </tbody>
</table>

---

## RouteInstruction Structure

Route instructions provide detailed navigation guidance, including coordinates, turn directions, distances, and time to waypoints.

<table>
    <thead>
        <tr>
            <th>Method</th>
            <th>Return Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>`coordinates`</td>
            <td>`Coordinates`</td>
            <td>Gets coordinates for this route instruction.</td>
        </tr>
        <tr>
            <td>`countryCodeISO`</td>
            <td>`String`</td>
            <td>Gets ISO 3166-1 alpha-3 country code for the navigation instruction.</td>
        </tr>
        <tr>
            <td>`exitDetails`</td>
            <td>`String`</td>
            <td>Gets exit route instruction text.</td>
        </tr>
        <tr>
            <td>`followRoadInstruction`</td>
            <td>`String`</td>
            <td>Gets textual description for follow road information.</td>
        </tr>
        <tr>
            <td>`realisticNextTurnImg`</td>
            <td>`AbstractGeometryImg`</td>
            <td>Gets image for the realistic turn information.</td>
        </tr>
        <tr>
            <td>`remainingTravelTimeDistance`</td>
            <td>`TimeDistance`</td>
            <td>Gets remaining travel time and distance until the destination</td>
        </tr>
        <tr>
            <td>`remainingTravelTimeDistanceToNextWaypoint`</td>
            <td>`TimeDistance`</td>
            <td>Gets remaining travel time and distance to the next waypoint.</td>
        </tr>
        <tr>
            <td>`roadInfo`</td>
            <td>`List<RoadInfo>`</td>
            <td>Gets road information.</td>
        </tr>
        <tr>
            <td>`roadInfoImg`</td>
            <td>`RoadInfoImg`</td>
            <td>Gets road information image.</td>
        </tr>
        <tr>
            <td>`signpostDetails`</td>
            <td>`SignpostDetails`</td>
            <td>Gets extended signpost details.</td>
        </tr>
        <tr>
            <td>`signpostInstruction`</td>
            <td>`String`</td>
            <td>Gets textual description for the signpost information.</td>
        </tr>
        <tr>
            <td>`timeDistanceToNextTurn`</td>
            <td>`TimeDistance`</td>
            <td>Gets distance and time to the next turn.</td>
        </tr>
        <tr>
            <td>`traveledTimeDistance`</td>
            <td>`TimeDistance`</td>
            <td>Gets traveled time and distance.</td>
        </tr>
        <tr>
            <td>`turnDetails`</td>
            <td>`TurnDetails`</td>
            <td>Gets full details for the turn.</td>
        </tr>
        <tr>
            <td>`turnImg`</td>
            <td>`Img`</td>
            <td>Gets image for the turn.</td>
        </tr>
        <tr>
            <td>`turnInstruction`</td>
            <td>`String`</td>
            <td>Gets textual description for the turn.</td>
        </tr>
        <tr>
            <td>`hasFollowRoadInfo`</td>
            <td>`bool`</td>
            <td>Checks if follow road information is available.</td>
        </tr>
        <tr>
            <td>`hasSignpostInfo`</td>
            <td>`bool`</td>
            <td>Checks if signpost information is available.</td>
        </tr>
        <tr>
            <td>`hasTurnInfo`</td>
            <td>`bool`</td>
            <td>Checks if turn information is available.</td>
        </tr>
        <tr>
            <td>`hasRoadInfo`</td>
            <td>`bool`</td>
            <td>Checks if road information is available.</td>
        </tr>
        <tr>
            <td>`isCommon`</td>
            <td>`bool`</td>
            <td>Checks if this instruction is of common type - has the same transport mode as the parent route</td>
        </tr>
        <tr>
            <td>`isExit`</td>
            <td>`bool`</td>
            <td>Checks if the route instruction is a main road exit instruction.</td>
        </tr>
        <tr>
            <td>`isFerry`</td>
            <td>`bool`</td>
            <td>Checks if the route instruction is on a ferry segment.</td>
        </tr>
        <tr>
            <td>`isTollRoad`</td>
            <td>`bool`</td>
            <td>Checks if the route instruction is on a toll road.</td>
        </tr>
    </tbody>
</table>

Distinguish between `NavigationInstruction` and `RouteInstruction`:

- `NavigationInstruction` - Real-time, turn-by-turn navigation based on current position (only during navigation or simulation)

- `RouteInstruction` - Route overview available immediately after calculation (instructions don't change during navigation)

---

## Related Classes 

### TimeDistance

The `TimeDistance` class provides time and distance details to important points of interest.

It differentiates between road types:

- **Restricted** - Non-public roads

- **Unrestricted** - Publicly accessible roads

| Field                      | Type    | Explanation                                                                                    |
|----------------------------|---------|------------------------------------------------------------------------------------------------|
| `unrestrictedTimeS`        | `int`   | Unrestricted time in seconds.                                                                  |
| `restrictedTimeS`          | `int`   | Restricted time in seconds.                                                                    |
| `unrestrictedDistanceM`    | `int`   | Unrestricted distance in meters.                                                               |
| `restrictedDistanceM`      | `int`   | Restricted distance in meters.                                                                 |
| `ndBeginEndRatio`          | `double`| Ratio representing the division of restricted time/distance between the begin and the end.     |
| `totalTimeS`               | `int`   | Total time in seconds (sum of unrestricted and restricted times).                              |
| `totalDistanceM`           | `int`   | Total distance in meters (sum of unrestricted and restricted distances).                       |
| `isEmpty`                  | `bool`  | Indicates whether the total time is zero.                                                      |
| `isNotEmpty`               | `bool`  | Indicates whether the total time is non-zero.                                                  |
| `hasRestrictedBeginEndDifferentiation` | `bool` | Indicates if the begin and end have differentiated restricted values based on the ratio.|
| `restrictedTimeAtBegin`    | `int`   | Restricted time allocated to the beginning, based on the ratio.                                |
| `restrictedTimeAtEnd`      | `int`   | Restricted time allocated to the end, based on the ratio.                                      |
| `restrictedDistanceAtBegin`| `int`  | Restricted distance allocated to the beginning, based on the ratio.                             |
| `restrictedDistanceAtEnd`  | `int`   | Restricted distance allocated to the end, based on the ratio.                                  |

### Signpost Details

Signposts near roadways indicate intersections and directions. The SDK provides realistic image renderings with additional information.

The `SignpostDetails` class provides:

| Member                | Type                       | Description                                                                                |
|:---------------------:|:--------------------------:|:------------------------------------------------------------------------------------------:|
| `backgroundColor`     | `Color`                    | Retrieves the background color of the signpost.                                            |
| `borderColor`         | `Color`                    | Retrieves the border color of the signpost.                                                |
| `textColor`           | `Color`                    | Retrieves the text color of the signpost.                                                  |
| `hasBackgroundColor`  | `bool`                     | Indicates whether the signpost has a background color.                                     |
| `hasBorderColor`      | `bool`                     | Indicates whether the signpost has a border color.                                         |
| `hasTextColor`        | `bool`                     | Indicates whether the signpost has a text color.                                           |
| `items`               | `List<SignpostItem>`       | Retrieves a list of `SignpostItem` elements associated with the signpost.                  |

Each `SignpostItem` provides:

| Member               | Type                          | Description                                                                              |
|:--------------------:|:-----------------------------:|:----------------------------------------------------------------------------------------:|
| `row`                | `int`                         | Retrieves the one-based row of the item. Zero indicates not applicable.                  |
| `column`             | `int`                         | Retrieves the one-based column of the item. Zero indicates not applicable.               |
| `connectionInfo`     | `SignpostConnectionInfo`      | Retrieves the connection type of the item (branch, towards, exit, invalid)               |
| `phoneme`            | `String`                      | Retrieves the phoneme assigned to the item if available, otherwise empty.                |
| `pictogramType`      | `SignpostPictogramType`       | Retrieves the pictogram type for the item (airport, busStation, parkingFacility, etc).   |
| `shieldType`         | `RoadShieldType`              | Retrieves the shield type for the item (county, state, federal, interstate, etc)         |
| `text`               | `String`                      | Retrieves the text assigned to the item, if available.                                   |
| `type`               | `SignpostItemType`            | Retrieves the type of the item (placeName, routeNumber, routeName, etc).                 |
| `hasAmbiguity`       | `bool`                        | Indicates if the item has ambiguity. Avoid using such items for TTS.                     |
| `hasSameShieldLevel` | `bool`                        | Indicates if the road code item has the same shield level as its road.                   |

### Turn Details

The `TurnDetails` class provides:

- **event** - Turn type enum (straight, right, left, lightLeft, lightRight, sharpRight, sharpLeft, roundaboutExitRight, and more)

- **abstractGeometryImg** - Abstract turn image (verify validity). Customize colors with `AbstractGeometryImageRenderSettings`

- **roundaboutExitNumber** - Roundabout exit number (if available)

#### Abstract Image vs. Turn Image

Compare images from `abstractGeometryImg` (left) and `turnImg` (right):

- **abstractGeometryImg** - Detailed intersection representation with customizable colors for theme alignment

- **turnImg** - Simplified turn schematic focusing on essential direction (no customization)

Use the `uid` getter to retrieve each image's unique identifier. Update the UI only when the ID changes to enhance navigation performance.

#### Customize Abstract Geometry Images

Use `AbstractGeometryImageRenderSettings` to customize render settings and colors:
```dart
 AbstractGeometryImageRenderSettings customizedRenderSettings = const AbstractGeometryImageRenderSettings(
    activeInnerColor: Colors.red,
    activeOuterColor: Colors.green,
    inactiveInnerColor: Colors.blue,
    inactiveOuterColor: Colors.yellow,
);
```

These settings produce:

---

## Toll Sections

The `TollSection` class represents a tolled route portion, defining start and end points (in meters from route start), cost, and currency.

| Member           | Type    | Description                                                                 |
|------------------|---------|-----------------------------------------------------------------------------|
| `startDistanceM` | `int`   | Distance in meters where the section starts (from route starting point).    |
| `endDistanceM`   | `int`   | Distance in meters where the section ends (from route starting point).      |
| `cost`           | `double`| Cost in the specified currency.                                             |
| `currency`       | `String`| Currency code, e.g. EUR, USD.                                               |

When cost data is unavailable, `cost` is 0 and `currency` is an empty string.

Get WGS coordinates of toll section start and end using `Route.getCoordinateOnRoute` with `startDistanceM` and `endDistanceM` values.

---

## Change Instruction Language

Route instruction texts follow the SDK language settings. See [Internationalization](/guides/get-started/internationalization) for details.

---
