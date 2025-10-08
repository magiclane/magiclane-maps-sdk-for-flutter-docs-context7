---
description: Documentation for Navigation Instructions
title: Navigation Instructions
---

# Navigation instructions

The Maps SDK for Flutter offers comprehensive real-time navigation guidance, providing detailed information on the current and upcoming route, including road details, street names, speed limits, and turn directions. It delivers essential data such as remaining travel time, distance to destination, and upcoming turn or road information, ensuring users receive accurate, timely instructions. Designed for both navigation and simulation scenarios, this feature enhances the overall user experience by supporting smooth and efficient route planning and execution.

The main class responsible for turn-by-turn live navigation. guidance is the ``NavigationInstruction`` class.

It is important to distinguish between ``NavigationInstruction`` and ``RouteInstruction``. ``NavigationInstruction`` offers real-time, turn-by-turn navigation based on the user's current position and is relevant only during navigation or simulation. In contrast, ``RouteInstruction`` provides an overview of the entire route available as soon as the route is calculated and the list of instructions do not change as the user navigates on the route.

## Instantiating navigation instructions

Navigation instructions cannot be directly instantiated. Instead, they must be provided by the SDK while navigating. For detailed guidance on how to navigate on routes, refer to the [Getting Started with Navigation Guide](/guides/navigation/get-started-navigation).

There are two main ways of getting a navigation instruction:

- `NavigationInstruction` instances can be obtained via the callback provided as parameters to the ``startNavigation`` and ``startSimulation`` methods. 

- The ``NavigationService`` class provides a ``getNavigationInstruction`` method which returns the currently available navigation instruction. Make sure navigation/simulation is active before using the method provided above.

## NavigationInstruction structure

<table>
 <thead>
   <tr>
     <th>Member</th>
     <th>Type</th>
     <th>Description</th>
   </tr>
 </thead>
 <tbody>
   <tr>
     <td>currentCountryCodeISO</td>
     <td>String</td>
     <td>Returns the ISO 3166-1 alpha-3 country code for the current navigation instruction. Empty string means no country.</td>
   </tr>
   <tr>
     <td>currentStreetName</td>
     <td>String</td>
     <td>Returns the current street name.</td>
   </tr>
   <tr>
     <td>currentStreetSpeedLimit</td>
     <td>double</td>
     <td>Returns the maximum speed limit on the current street in meters per second. Returns 0 if not available.</td>
   </tr>
   <tr>
     <td>driveSide</td>
     <td>DriveSide</td>
     <td>Returns the drive side flag of the current traveled road.</td>
   </tr>
   <tr>
     <td>hasNextNextTurnInfo</td>
     <td>bool</td>
     <td>Returns true if next-next turn information is available.</td>
   </tr>
   <tr>
     <td>hasNextTurnInfo</td>
     <td>bool</td>
     <td>Returns true if next turn information is available.</td>
   </tr>
   <tr>
     <td>instructionIndex</td>
     <td>int</td>
     <td>Returns the index of the current route instruction on the current route segment.</td>
   </tr>
   <tr>
     <td>laneImg</td>
     <td>LaneImg</td>
     <td>Returns a customizable image representation of current lane configuration. The user is responsabile to verify if the image is valid.</td>
   </tr>
   <tr>
     <td>navigationStatus</td>
     <td>NavigationStatus</td>
     <td>Returns the navigation/simulation status.</td>
   </tr>
   <tr>
     <td>nextCountryCodeISO</td>
     <td>String</td>
     <td>Returns the ISO 3166-1 alpha-3 country code for the next navigation instruction.</td>
   </tr>
   <tr>
     <td>nextNextStreetName</td>
     <td>String</td>
     <td>Returns the next-next street name.</td>
   </tr>
   <tr>
     <td>nextNextTurnDetails</td>
     <td>TurnDetails</td>
     <td>Returns the full details for the next-next turn. Used for customizing turn display in UI.</td>
   </tr>
   <tr>
     <td>nextNextTurnImg</td>
     <td>Img</td>
     <td>Returns a simplified schematic image of the next-next turn. The user is responsabile to verify if the image is valid.</td>
   </tr>
   <tr>
     <td>nextNextTurnInstruction</td>
     <td>String</td>
     <td>Returns the textual description for the next-next turn.</td>
   </tr>
   <tr>
     <td>getNextSpeedLimitVariation</td>
     <td>NextSpeedLimit</td>
     <td>Returns the next speed limit variation within specified check distance.</td>
   </tr>
   <tr>
     <td>nextStreetName</td>
     <td>String</td>
     <td>Returns the next street name.</td>
   </tr>
   <tr>
     <td>nextTurnDetails</td>
     <td>TurnDetails</td>
     <td>Returns the full details for the next turn. Used for customizing turn display in UI.</td>
   </tr>
   <tr>
     <td>nextTurnImg</td>
     <td>Img</td>
     <td>Returns a simplified schematic image of the next turn. The user is responsabile to verify if the image is valid.</td>
   </tr>
   <tr>
     <td>nextTurnInstruction</td>
     <td>String</td>
     <td>Returns the textual description for the next turn.</td>
   </tr>
   <tr>
     <td>remainingTravelTimeDistance</td>
     <td>TimeDistance</td>
     <td>Returns the remaining travel time in seconds and distance in meters.</td>
   </tr>
   <tr>
     <td>remainingTravelTimeDistanceToNextWaypoint</td>
     <td>TimeDistance</td>
     <td>Returns the remaining travel time in seconds and distance in meters to the next waypoint.</td>
   </tr>
   <tr>
     <td>currentRoadInformation</td>
     <td>List&lt;RoadInfo&gt;</td>
     <td>Returns the current road information list.</td>
   </tr>
   <tr>
     <td>nextRoadInformation</td>
     <td>List&lt;RoadInfo&gt;</td>
     <td>Returns the next road information list.</td>
   </tr>
   <tr>
     <td>nextNextRoadInformation</td>
     <td>List&lt;RoadInfo&gt;</td>
     <td>Returns the next-next road information list.</td>
   </tr>
   <tr>
     <td>segmentIndex</td>
     <td>int</td>
     <td>Returns the index of the current route segment.</td>
   </tr>
   <tr>
     <td>signpostDetails</td>
     <td>SignpostDetails</td>
     <td>Returns the extended signpost details.</td>
   </tr>
   <tr>
     <td>signpostInstruction</td>
     <td>String</td>
     <td>Returns the textual description for the signpost information.</td>
   </tr>
   <tr>
     <td>timeDistanceToNextNextTurn</td>
     <td>TimeDistance</td>
     <td>Returns the time (seconds) and distance (meters) to the next-next turn. Returns values for next turn if no next-next turn available.</td>
   </tr>
   <tr>
     <td>timeDistanceToNextTurn</td>
     <td>TimeDistance</td>
     <td>Returns the time (seconds) and distance (meters) to the next turn.</td>
   </tr>
   <tr>
     <td>traveledTimeDistance</td>
     <td>TimeDistance</td>
     <td>Returns the traveled time in seconds and distance in meters.</td>
   </tr>
 </tbody>
</table>

## Turn details

### Next turn details

The following snippet shows how to extract detailed instructions for the next turn along the route. It’s typically used in the navigation UI to show users the upcoming maneuver. You may also use this to provide turn-by-turn instructions with images or detailed text for navigation display:
```dart
// If hasNextTurnInfo is false some details are not available
bool hasNextTurnInfo = navigationInstruction.hasNextTurnInfo;

if (hasNextTurnInfo) {
    // The next turn instruction
    String nextTurnInstruction = navigationInstruction.nextTurnInstruction;

    // The next turn details
    TurnDetails turnDetails = navigationInstruction.nextTurnDetails;

    // Turn event type (continue straight, turn right, turn left, etc.)
    TurnEvent event = turnDetails.event;

    // The image representation of the abstract geometry
    Uint8List? abstractGeometryImage = turnDetails.getAbstractGeometryImage(
        size: Size(300, 300),
    );

    // The image representation of the next turn
    Uint8List? turnImage = navigationInstruction.getNextTurnImage(size: Size(300, 300));

    // Roundabout exit number (-1 if not a roundabout)
    int roundaboutExitNumber = turnDetails.roundaboutExitNumber;
}
```

See the [TurnDetails](/guides/core/routes#turn-details) guide for more details about the fields within the `TurnDetails` class.

### Next next turn details

Details about the subsequent turn (i.e., the turn following the next one) can be crucial for certain use cases, such as providing a preview of upcoming maneuvers. These details can be accessed in a similar manner:
```dart
// If hasNextNextTurnInfo is false some details are not available
bool hasNextNextTurnInfo = navigationInstruction.hasNextNextTurnInfo;

if (hasNextNextTurnInfo) {
    String nextNextTurnInstruction = navigationInstruction.nextNextTurnInstruction;

    TurnDetails nextNextTurnDetails = navigationInstruction.nextNextTurnDetails;

    Uint8List? nextNextTurnImage = navigationInstruction.getNextNextTurnImage(size: Size(300, 300));
}
```

The ``hasNextNextTurnInfo`` might be false if the next instruction is the destination.
The same operations discussed earlier for the next turn details can also be applied to the subsequent turn details.

## Street information

### Current street information

The following snippet shows how to get information about the current road:
```dart
// Current street name
String currentStreetName = navigationInstruction.currentStreetName;

// Road info related to the current road
List<RoadInfo> currentRoadInfo = navigationInstruction.currentRoadInformation;

// Country ISO code
String countryCode = navigationInstruction.currentCountryCodeISO;

// The drive direction (left or right)
DriveSide driveDirection = navigationInstruction.driveSide;
```

It is important to note that some streets may not have an assigned name. In such cases, ``currentStreetName`` will return an empty string. The ``RoadInfo`` class offers additional details, including the road name and shield type, which correspond to the official codes or names assigned to a road.

For example, the `currentStreetName` might return "Bloomsbury Street," while the `roadname` field in the associated RoadInfo instance could provide the official designation, such as "A400." This distinction ensures comprehensive road identification.

### Next & next next street information

Information about the next street, as well as the street following it (next-next street), can also be retrieved. These details include the street name, type, and other associated metadata, enabling enhanced navigation and situational awareness:
```dart
// Street name
String nextStreetName = navigationInstruction.nextStreetName;
String nextNextStreetName = navigationInstruction.nextNextStreetName;

// Road info
List<RoadInfo> nextRoadInformation = navigationInstruction.nextRoadInformation;
List<RoadInfo> nextNextRoadInformation = navigationInstruction.nextNextRoadInformation;

// Next country iso code
String nextCountryCodeISO = navigationInstruction.nextCountryCodeISO;
```

The fields associated with these streets retain the same meanings as discussed earlier about the current street.

Ensure that ``hasNextTurnInfo`` and ``hasNextNextTurnInfo`` are true before attempting to access the respective fields. This verification prevents errors and ensures the availability of reliable data for the requested information.

## Speed limit information

The NavigationInstruction class not only provides information about the current road's speed limit but also offers details about upcoming speed limits within a specified distance, assuming the user adheres to the recommended navigation route.

The following snippet demonstrates how to retrieve these details and handle various scenarios appropriately:
```dart
// The current street speed limit in m/s (0 if not available)
double currentStreetSpeedLimit = navigationInstruction.currentStreetSpeedLimit;

// Get next speed limit in 500 meters
NextSpeedLimit nextSpeedLimit = navigationInstruction.getNextSpeedLimitVariation(checkDistance: 500);
// Coordinates where next speed limit changes
Coordinates coordinatesWhereSpeedLimitChanges = nextSpeedLimit.coords;
// Distance to where the next speed limit changes
int distanceToNextSpeedLimitChange = nextSpeedLimit.distance;
// Value of the next speed limit (m/s)
double nextSpeedLimitValue = nextSpeedLimit.speed;

if (distanceToNextSpeedLimitChange == 0 && nextSpeedLimitValue == 0) {
    showSnackbar("The speed limit does not change within the specified interval");
} else if (nextSpeedLimitValue == 0) {
    showSnackbar(
        "The speed limit changes in the specified interval but the value is not available");
} else {
    showSnackbar(
        "The next speed limit changes to $nextSpeedLimitValue m/s in $distanceToNextSpeedLimitChange meters");
}
```

## Lane image

The lane image can be used to more effectively illustrate the correct lane for upcoming turns, providing clearer guidance:
```dart
final LaneImg laneImage = navigationInstruction.laneImage;
final Uint8List? laneImageData = laneImage.getRenderableImageBytes(size: Size(500, 300), format: ImageFileFormat.png);
```

Below is an example of a rendered lane image:

## Change the language of the instructions

The texts used in navigation instructions and related classes follow the language set in the SDK. See [the internationalization guide](../2_get-started/05_internationalization.mdx) for more details.
