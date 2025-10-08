---
description: Documentation for Roadblocks
title: Roadblocks
---

# Roadblocks

A roadblock is a user-defined restriction applied to a specific road segment or geographic area, used to reflect traffic disruptions such as construction, closures, or areas to avoid.

It influences route planning by marking certain paths or zones as unavailable for navigation.

Roadblocks can be **path-based** (defined by a sequence of coordinates) or **area-based** (covering a geographic region), and may be either **temporary** or **persistent**, depending on their intended duration. Persistent roadblocks remain after a SDK uninitialization. Temporary roadblocks are short-lived.

The primary entity responsible for representing roadblocks is the `TrafficEvent` class. Check the [Traffic Events guide](../core/traffic-events) for more details.
Roadblocks are mainly managed through the `TrafficService` class.

While some roadblocks are provided in real time by online data from Magic Lane servers, users can also define their own **user roadblocks** to customize routing behavior.

If the applied style includes traffic data and traffic display is enabled (`MapViewPreferences.setTrafficVisibility` is set to true), a visual indication of the blocked portion will appear on the map, highlighted in red.

Adding/removing user roadblocks affects only the current user and does not impact other users' routes. To create reports that are visible to all users, refer to the [Social Reports guide](../social-reports).

## Configure the traffic service

Traffic behavior can be customized through the `TrafficPreferences` instance, accessible via the `TrafficService` class. The `TrafficPreferences` class provides the `useTraffic` property, which defines how traffic data should be applied during routing and navigation.

The `TrafficUsage` enum offers the following configuration options:

| Value     | Description                                                   |
|-----------|---------------------------------------------------------------|
| `none`    | Disables all traffic data usage.                              |
| `online`  | Uses both online and offline traffic data (default setting).  |
| `offline` | Uses only offline traffic data, including user-defined roadblocks. |

For example, in order to set allow only offline usage the following line can be used:
```dart
TrafficService.preferences.useTraffic = TrafficUsage.offline;
```

## Add a temporary user roadblock while in navigation

A roadblock can be added to bypass a portion of the route for a specified distance. Once the roadblock is applied, the route will be recalculated, and the updated route will be returned via the `onRouteUpdated` callback provided to either the `startNavigation` or `startSimulation` method.

The snippet below will add a roadblock of 100 meters starting in 10 meters:
```dart
NavigationService.setNavigationRoadBlock(100, startDistance: 10);
```

Roadblocks added through the `setNavigationRoadBlock` method provided by the `NavigationService` only affect the ongoing navigation.

## Check if a geographic position has traffic information

The `getOnlineServiceRestrictions` method can be used. It takes a `Coordinates` object as argument and returns a `TrafficOnlineRestrictions` enum.

For example, in order to check if traffic events are available for a certain geographic position:
```dart
Coordinates coords = Coordinates.fromLatLong(50.108, 8.783);
TrafficOnlineRestrictions restriction = TrafficService.getOnlineServiceRestrictions(coords);
```

The `TrafficOnlineRestrictions` enum provides the following values:

- `none`: No restrictions are in place; online traffic is available.

- `settings`:  Online traffic is disabled in the `TrafficPreferences` object.

- `connection`:  No internet connection is available.

- `networkType`:  Not allowed on extra charged networks (e.g., roaming).

- `providerData`:  Required provider data is missing.

- `worldMapVersion`:  The world map version is outdated and incompatible. Please update the road map.

- `diskSpace`:  Insufficient disk space to download or store traffic data.

- `initFail`:  Failed to initialize the traffic service.

## Add a user-defined persistent roadblock

To add a persistent user-defined roadblock, the user must provide the following:

- **startTime** : the timestamp indicating when the roadblock becomes active.

- **expireTime** : the timestamp indicating when the roadblock is no longer in effect.

- **transportMode** : the specific mode of transport affected by the roadblock.

- **id** : a unique string ID for the roadblock.

- **coords/area** : either:

  - a list of coordinates (for path-based roadblocks), or

  - a geographic area (for area-based roadblocks).

When a user-defined roadblock is added, it will affect routing and navigation between the specified **startTime** and **expireTime**. Once the **expireTime** is reached, the roadblock is automatically removed without any user intervention.

The following conditions apply when adding a roadblock:

- If roadblocks are disabled in the `TrafficPreferences` object, the addition will fail with the `GemError.activation` code.

- If a roadblock already exists at the same location where the user attempts to add a new one, the operation will fail with the `GemError.exist` code.

- If the input parameters are invalid (e.g., **expireTime** is later than **startTime**, missing **id**, or invalid coordinates/area object), the addition will fail with the `GemError.invalidInput` code.

- If a roadblock with the same **id** already exists, the addition will fail with the `GemError.inUse` code.

### Add a area-based persistent roadblock

The `addPersistentRoadblockByArea` method is used to add **area-based** user roadblocks. It accepts a `GeographicArea` object which represents the area to be avoided.

The method returns:

- If the addition is successful, the method returns the newly created `TrafficEvent` instance along with the `GemError.success` code.

- If the addition fails, the method returns `null` and an appropriate `GemError` code, indicating the reason for the failure.

For example, adding a area-based user-defined persistent roadblock on a given area, starting from now and available for 1 hour which affects cars can be done in the following way:
```dart
final area = RectangleGeographicArea(
    topLeft: Coordinates.fromLatLong(46.764942, 7.122563),
    bottomRight: Coordinates.fromLatLong(46.762031, 7.127992),
);

final (TrafficEvent?, GemError) result = TrafficService.addPersistentRoadblockByArea(
    area: area,
    startTime: DateTime.now(),
    expireTime: DateTime.now().add(Duration(hours: 1)),
    transportMode: RouteTransportMode.car,
    id: 'test_id',
);

if (result.$2 == GemError.success) {
    print("The addition was successful");
    TrafficEvent event = result.$1!;
} else {
    print("The addition failed with error code ${result.$2}");
}
```

### Add a path-based persistent roadblock

The `addPersistentRoadblockByCoordinates` method is used to add **path-based** user roadblocks. It accepts a list of `Coordinate` objects and supports two modes of operation:

- **Single Coordinate**: Defines a **point-based** roadblock. This may result in two roadblocks being created—one for each travel direction.

- **Multiple Coordinates**: Defines a **path-based** roadblock, starting at the first coordinate and ending at the last. This is used to restrict access along a specific road segment.

For example, adding a path-based user-defined persistent roadblock on both sides of the matching road, starting from now and available for 1 hour which affects cars can be done in the following way:
```dart
final coords = [Coordinates.fromLatLong(405.847994, 24.956233)];

(TrafficEvent?, GemError) result = TrafficService.addPersistentRoadblockByCoordinates(
    coords: coords,
    startTime: DateTime.now(),
    expireTime: DateTime.now().add(Duration(hours: 1)),
    transportMode: RouteTransportMode.car,
    id: 'test_id',
);

if (result.$2 == GemError.success) {
    print("The addition was successful");
    TrafficEvent event = result.$1!;
} else {
    print("The addition failed with error code ${result.$2}");
}
```

The method returns the result in a similar way to the `addPersistentRoadblockByArea` method.

In addition to the scenarios described above, the `addPersistentRoadblockByCoordinates` method may also fail in the following cases:

- **No Suitable Road Found**: If a valid road cannot be identified at the specified coordinates, or if no road data (online or offline) is available for the given location, the method will return `null` along with the `GemError.notFound` error code.

- **Route Computation Failed**: If multiple coordinates are provided but a valid route cannot be computed between them, the method will return `null` and the `GemError.noRoute` error code.

### Add an anti-area persistent roadblock

If a region contains a persistent roadblock, the user may wish to whitelist a specific sub-area within the larger restricted zone to allow routing and navigation through that portion. This can be achieved using the `addAntiPersistentRoadblockByArea` method, which accepts the same arguments as the `addPersistentRoadblockByArea` method described above.

This functionality enables fine-grained control over blocked regions by allowing exceptions within otherwise restricted areas.

## Get all user-defined persistent roadblocks

The `persistentRoadblocks` getter provided by the `TrafficService` provides the list of persistent roadblocks.

The following snippet iterates through all persistent roadblocks—both path-based and area-based—and prints their unique identifiers.
```dart
List<TrafficEvent> roadblocks = TrafficService.persistentRoadblocks;

for (final roadblock in roadblocks){
    print(roadblock.description);
}
```

All user-defined roadblocks that are currently active or scheduled to become active are returned. Expired roadblocks are automatically removed.

## Get user-defined persistent roadblocks

To get both path-based and area-based roadblocks if the identifier is known use the `getPersistentRoadblock` method.
This method takes the identifier string as argument and returns null if the event could not be found or the event if it exists.
```dart
TrafficEvent? event = TrafficService.getPersistentRoadblock("unique_id");

if (event != null){
    print("Event was found");
} else {
    print("Event does not exist");
}
```

## Remove user-defined roadblocks

### Remove persistent user-defined roadblock by id

Use the `removePersistentRoadblockById` method to remove a roadblock if the identifier is known.
```dart
GemError error = TrafficService.removePersistentRoadblockById("identifier");
if (error == GemError.succss){
    print("Removal succeded");
} else {
    print("Removal failed with error code $error");
}
```

The method returns `GemError.success` if the roadblock was removed and `GemError.notFound` if no roadblock was found with the given id.
This method work both for path-based and area-based roadblocks.

### Remove persistent user-defined roadblock by coordinates

Use the `removePersistentRoadblockByCoordinates` method to remove a roadblock to a remove path-based roadblocks by providing the method with the *first* coordinate of the roadblock to be removed.
```dart
GemError error = TrafficService.removePersistentRoadblockByCoordinates(coords);
if (error == GemError.success){
    print("Removal succeded");
} else {
    print("Removal failed with error code $error");
}
```

The method returns `GemError.success` if the roadblock was removed and `GemError.notFound` if no roadblock was found starting with the given coordinate.

### Remove user-defined roadblock given the TrafficEvent

If the `TrafficEvent` instance is available, it can be removed using the `removeUserRoadblock` method:
```dart
TrafficEvent event = ...
TrafficService.removeUserRoadblock(event);
```

This method can be used for both persistent and non-persistent roadblocks.

### Remove all user-defined persistent roadblocks

Use the `removeAllPersistentRoadblocks` method to delete all existing user-defined roadblocks.

## Get preview of a path-based user-defined roadblock

Before adding a persistent user roadblock, the user can preview the path using an intermediary list of coordinates generated between two positions. This functionality is provided by the `getPersistentRoadblockPathPreview` method, which helps visualize the intended roadblock on the map.

This method takes as arguments:

- `UserRoadblockPathPreviewCoordinate from` - The starting point of the roadblock. Can be obtained:

  - Can be constructed from a `Coordinates` object using the `.fromCoordinates()` factory constructor.  

  - Also returned by the `getPersistentRoadblockPathPreview` method to allow daisy-chaining multiple segments.

- `Coordinates to` - The ending point of the roadblock.

- `RouteTransportMode transportMode` - The transport mode (e.g., car, bicycle, pedestrian) to be used for the roadblock preview and calculation.

The method returns a tuple containing:

- `List<Coordinates>` - A list of intermediate coordinates forming the preview path. This list can be used to render a polyline or marker path on the map.

- `UserRoadblockPathPreviewCoordinate` - The updated "end" coordinate, which can be reused as the `from` argument to preview or chain additional segments.

- `GemError` - Error code of the operation. This may include the same error codes returned by `addPersistentRoadblockByCoordinates`. The rest of the return values are not valid if the error is not success.

For example, in oder to get the preview of a user-defined path-based roadblock between two `Coordinate` objects:
```dart
Coordinates startCoordinates = ...
Coordinates endCoordinates = ...

UserRoadblockPathPreviewCoordinate previewStart = UserRoadblockPathPreviewCoordinate.fromCoordinates(startCoordinates);

final (coordinates, newPreviewStart, previewError) =
    TrafficService.getPersistentRoadblockPathPreview(
    from: previewStart,
    to: endCoordinates,
    transportMode: RouteTransportMode.car,
);
previewStart = newPreviewStart;

if (previewError != GemError.success) {
    print("Error $previewError during preview calculation");
} else {
    // Draw the path on the UI
    Path previewPath = Path.fromCoordinates(coordinates);
    controller.preferences.paths.add(previewPath);

    // If the user is happy with the roadblock preview,
    // the roadblock can be added using addPersistentRoadblockByCoordinates
}
```

## Persistent roadblock listener

The Magic Lane SDK for Flutter also allows users to register for notifications related to persistent roadblocks. These notifications are triggered in the following cases:

- When a roadblock's `startTime` becomes greater than the current time - via the `onRoadblocksActivated` callback

- When a roadblock's `endTime` becomes less than the current time - via the `onRoadblocksExpired` callback

These callback provide the activated/expired `List<TrafficEvent>`.

Use the constructor to instanciate an object of the `PersistentRoadblockListener` class:
```dart
final PersistentRoadblockListener listener = PersistentRoadblockListener(
    onRoadblocksActivated: (List<TrafficEvent> events) {
        // Do something with the events
    },
    onRoadblocksExpired: (List<TrafficEvent> events) {
        // Do something with the events
    },
);
```

The `onRoadblocksActivated` and `onRoadblocksExpired` callbacks can also be registered/changed via the `registerOnRoadblocksExpired` and `registerOnRoadblocksActivated` methods.

Register the listener using the `persistentRoadblockListener` static setter of the `TrafficService` class:
```dart
TrafficService.persistentRoadblockListener = listener;
```


