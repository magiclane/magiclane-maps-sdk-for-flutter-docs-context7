---
description: Documentation for Routes
title: Routes
---

# Routes

A Route usually represents a navigable path between two or more landmarks (waypoints). It includes data such as distance, estimated time, and navigation instructions.

Routes can be computed in different ways:

- Based on 2 or more intermediary landmarks (waypoints) - route is navigable.

- Over-track routes (based on a predefined `path`, which could come from a GPX file but is not limited to this) are navigable.

- Route ranges: These routes are **not** navigable and do not have segments or instructions.

A navigable route consists of one or more segments. Each segment represents the portion of the route between two consecutive waypoints and includes its own set of route instructions.

## Instantiating Routes

Routes cannot be instantiated directly. Instead, they must be computed based on a predefined list of landmarks. For detailed guidance on how to calculate routes, refer to the [Getting Started with Routing Guide](/guides/routing/get-started-routing).

Calculating a route does not automatically display it on the map. Refer to the [Display markers guide](/guides/maps/display-map-items/display-markers) for detailed instructions on how to display one or more routes.

## Route specializations

The Maps SDK for Flutter supports multiple routes types, each tailored for specific use cases and implemented through dedicated classes:

1. **Normal Routes** - Standard routes computed for typical navigation scenarios.

2. **Public Transport (PT) Routes** - Routes calculated using a public transport mode, providing additional details such as frequency, ticket purchase information, and other public transport-specific data.

3. **Over-Track (OT) Routes** - Routes generated based on a predefined path, such as those derived from GPX files or routes drawn with the finder.

4. **Electric Vehicle (EV) Routes**  - Not fully implemented at the moment. Are designed for EV-specific needs, including charging station information and related details.

### Specific Classes

Each route type is associated with specific classes that offer functionality suited to its requirements:

| **Route Type**            | **Route Class** | **Segment Class**      | **Instruction Class**     |
|---------------------------|-----------------|------------------------|---------------------------|
| Normal Route              | `Route`         | `RouteSegment`         | `RouteInstruction`        |
| Public Transport Route    | `PTRoute`       | `PTRouteSegment`       | `PTRouteInstruction`      |
| Over-Track Route          | `OTRoute`       | Not Available          | Not Available             |
| Electric Vehicle Route    | `EVRoute`       | `EVRouteSegment`       | `EVRouteInstruction`      |

The specific classes extend the functionality of the base classes `RouteBase`, `RouteSegmentBase`, `RouteInstructionBase` that provide the common features for each type of entity.

## Route structure

The most important route characteristics are:

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

## RouteSegment structure

A route segment represents the portion of a route between two consecutive waypoints. For public transport routes, segments are further categorized as either pedestrian sections or public transit sections, depending on the mode of travel within that segment.  

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

## RouteInstruction structure

A route instruction provides detailed guidance for navigation, offering various methods to retrieve specific information about each maneuver the user needs to make, such as coordinates, turn directions, distances and time to next waypoints.

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

It is important to distinguish between ``NavigationInstruction`` and ``RouteInstruction``. ``NavigationInstruction`` offers real-time, turn-by-turn navigation based on the user's current position and is relevant only during navigation or simulation. In contrast, ``RouteInstruction`` provides an overview of the entire route available as soon as the route is calculated and the list of instructions do not change as the user navigates on the route.

## Other classes 

### Time distance

The TimeDistance class is used to get details about the time and distance to/from certain important points of interests.

The TimeDistance class differentiates between unrestricted and restricted road portions. Restricted segments refer to non-public roads, while unrestricted represent publicly accessible roads:

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

### Signpost details

Signposts near roadways typically indicate intersections and directions to various destinations. The Maps SDK for Flutter provides realistic image renderings of these signposts, along with additional relevant information.

Below is an example of a rendered signpost details image:

The ``SignpostDetails`` class also provides properties such as:

| Member                | Type                       | Description                                                                                |
|:---------------------:|:--------------------------:|:------------------------------------------------------------------------------------------:|
| `backgroundColor`     | `Color`                    | Retrieves the background color of the signpost.                                            |
| `borderColor`         | `Color`                    | Retrieves the border color of the signpost.                                                |
| `textColor`           | `Color`                    | Retrieves the text color of the signpost.                                                  |
| `hasBackgroundColor`  | `bool`                     | Indicates whether the signpost has a background color.                                     |
| `hasBorderColor`      | `bool`                     | Indicates whether the signpost has a border color.                                         |
| `hasTextColor`        | `bool`                     | Indicates whether the signpost has a text color.                                           |
| `items`               | `List<SignpostItem>`       | Retrieves a list of `SignpostItem` elements associated with the signpost.                  |

Each ``SignpostItem`` has the following properties:

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

The field ``nextTurnInstruction`` provides a instruction in text format, suitable for displaying on UI. Please use the ``onTextToSpeechInstruction`` callback for getting a instruction suitable for text-to-speech.

### Turn Details

The ``TurnDetails`` class provides details such as:

- **event**: enum containing the turn type. Values for this field include straight, right, left, lightLeft, lightRight, sharpRight, sharpLeft, roundaboutExitRight, roundabout, roundRight, roundLeft, infoGeneric, driveOn, exitNr, exitLeft, exitRight, stayOn and more.

- **abstractGeometryImg**: abstract image of the turn (the user needs to verify if the image is valid). The colors might be further personalized with ``AbstractGeometryImageRenderSettings``.

- **roundaboutExitNumber**: the roundabout exist number if it exists.

#### Turn details abstract image vs. turn image:

The difference between images obtained via ``abstractGeometryImg`` (left) and ``turnImg`` (right) is shown below:

- **abstractGeometryImg**: Provides a detailed representation of the entire intersection, offering a comprehensive view of the turn geometry. This image can be customized, including options to adjust various colors for better visual alignment with the application's theme.

- **turnImg**: Delivers a simplified schematic of the upcoming turn, focusing solely on the essential turn direction. Unlike abstractGeometryImg, this method does not support customization.

Each image is assigned a unique identifier. You can use the ``uid`` getter of the images classes to retrieve this id and update the image in the UI only when the ID changes. This strategy can significantly enhance performance during navigation, especially in scenarios where frequent updates might otherwise impact efficiency.

#### Abstract geometry image customization

The ``AbstractGeometryImageRenderSettings`` allows for abstract geometry render settings customization, with the possibility of specifying the colors, improving the overall user experience:
```dart
 AbstractGeometryImageRenderSettings customizedRenderSettings = const AbstractGeometryImageRenderSettings(
    activeInnerColor: Colors.red,
    activeOuterColor: Colors.green,
    inactiveInnerColor: Colors.blue,
    inactiveOuterColor: Colors.yellow,
);
```

The render settings from above will create the following image when used:

## Toll sections

The `TollSection` class represents a tolled portion of a route.
It defines where the toll segment starts and ends (measured in meters from the beginning of the route), along with the toll cost and currency.

| Member           | Type    | Description                                                                 |
|------------------|---------|-----------------------------------------------------------------------------|
| `startDistanceM` | `int`   | Distance in meters where the section starts (from route starting point).    |
| `endDistanceM`   | `int`   | Distance in meters where the section ends (from route starting point).      |
| `cost`           | `double`| Cost in the specified currency.                                             |
| `currency`       | `String`| Currency code, e.g. EUR, USD.                                               |

If no data for the cost of the section is available then the cost field is 0 and the currency field is empty string.

In order to get the WGS Coordinates of the start and end of a `TollSection`, you can use the `Route.getCoordinateOnRoute` method, passing the `startDistanceM` and `endDistanceM` values respectively.

## Change the language of the instructions

The texts used in route instructions and related classes follow the language set in the SDK. See [the internationalization guide](../2_get-started/05_internationalization.mdx) for more details.
