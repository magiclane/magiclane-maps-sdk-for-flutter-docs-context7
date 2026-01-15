---
description: Documentation for Navigation Instructions
title: Navigation Instructions
---

# Navigation instructions

The Maps SDK for Flutter provides real-time navigation guidance with detailed route information, including road details, street names, speed limits, and turn directions. You receive essential data such as remaining travel time, distance to destination, and upcoming maneuvers.

The main class responsible for turn-by-turn navigation guidance is the `NavigationInstruction` class.

Distinguish between `NavigationInstruction` and `RouteInstruction`. `NavigationInstruction` offers real-time, turn-by-turn guidance based on your current position and is relevant only during active navigation or simulation. In contrast, `RouteInstruction` provides an overview of the entire route available immediately after calculation, with instructions that remain static throughout navigation.

---

## Get navigation instructions

You cannot directly instantiate navigation instructions. The SDK provides them during active navigation. For detailed guidance, see the [Getting Started with Navigation Guide](/guides/navigation/get-started-navigation).

There are two ways to get navigation instructions:

- **Via callback** - `NavigationInstruction` instances are provided through callbacks in the `startNavigation` and `startSimulation` methods

- **Via service** - The `NavigationService` class provides a `getNavigationInstruction` method that returns the current navigation instruction

Ensure navigation or simulation is active before calling `getNavigationInstruction`.

---

## Understand the structure

The `NavigationInstruction` class contains the following members:

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

The `nextTurnInstruction` field provides text suitable for UI display. Use the `onTextToSpeechInstruction` callback for text-to-speech output.

---

## Access turn details

### Get next turn details

Extract detailed instructions for the next turn along the route. Use this information in your navigation UI to display upcoming maneuvers with images or detailed text:
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

See the [TurnDetails](/guides/core/routes#turn-details) guide for more details.

### Get next-next turn details

Access details about the turn following the next one to provide a preview of upcoming maneuvers:
```dart
// If hasNextNextTurnInfo is false some details are not available
bool hasNextNextTurnInfo = navigationInstruction.hasNextNextTurnInfo;

if (hasNextNextTurnInfo) {
    String nextNextTurnInstruction = navigationInstruction.nextNextTurnInstruction;

    TurnDetails nextNextTurnDetails = navigationInstruction.nextNextTurnDetails;

    Uint8List? nextNextTurnImage = navigationInstruction.getNextNextTurnImage(size: Size(300, 300));
}
```

The `hasNextNextTurnInfo` returns false if the next instruction is the destination.

You can apply the same operations from next turn details to next-next turn details.

---

## Get street information

### Access current street details

Retrieve information about the current road:
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

Some streets may not have an assigned name. In such cases, `currentStreetName` returns an empty string.

The `RoadInfo` class provides additional details, including the road name and shield type, which correspond to official road codes.

For example, `currentStreetName` might return "Bloomsbury Street," while the `roadname` field in the associated `RoadInfo` instance provides the official designation, such as "A400."

### Access next & next-next street details

Retrieve information about the next street and the street following it:
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

These fields have the same meanings as the current street fields.

Ensure `hasNextTurnInfo` and `hasNextNextTurnInfo` are true before accessing the respective fields. This prevents errors and ensures data availability.

---

## Get speed limit information

The `NavigationInstruction` class provides information about the current road's speed limit and upcoming speed limits within a specified distance.

Retrieve speed limit details and handle various scenarios:
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

---

## Display lane guidance

Use the lane image to illustrate the correct lane for upcoming turns:
```dart
final LaneImg laneImage = navigationInstruction.laneImage;
final Uint8List? laneImageData = laneImage.getRenderableImageBytes(size: Size(500, 300), format: ImageFileFormat.png);
```

---

## Change instruction language

Navigation instruction texts follow the language set in the SDK. See [the internationalization guide](/guides/get-started/internationalization) for more details.
