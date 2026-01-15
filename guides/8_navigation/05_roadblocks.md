---
description: Documentation for Roadblocks
title: Roadblocks
---

# Roadblocks

This guide explains how to add, manage, and remove roadblocks to customize route planning and navigation.

---

A **roadblock** is a user-defined restriction applied to a specific road segment or geographic area. It reflects traffic disruptions such as construction, closures, or areas to avoid, and influences route planning by marking certain paths or zones as unavailable for navigation.

Roadblocks can be **path-based** (defined by a sequence of coordinates) or **area-based** (covering a geographic region). They may be **temporary** (short-lived) or **persistent** (remain after SDK uninitialization), depending on their intended duration.

The `TrafficEvent` class represents roadblocks. Check the [Traffic Events guide](../core/traffic-events) for more details. Roadblocks are managed through the `TrafficService` class.

While some roadblocks are provided in real time by online data from Magic Lane servers, you can also define your own **user roadblocks** to customize routing behavior.

If the applied style includes traffic data and traffic display is enabled (`MapViewPreferences.setTrafficVisibility` is set to true), a visual indication of the blocked portion will appear on the map, highlighted in red.

Adding or removing user roadblocks affects only the current user and does not impact other users' routes. To create reports that are visible to all users, refer to the [Social Reports guide](../social-reports).

---

## Configure traffic service

Traffic behavior can be customized through the `TrafficPreferences` instance, accessible via the `TrafficService` class. The `useTraffic` property defines how traffic data should be applied during routing and navigation.

The `TrafficUsage` enum offers the following options:

| Value     | Description                                                   |
|-----------|---------------------------------------------------------------|
| `none`    | Disables all traffic data usage                              |
| `online`  | Uses both online and offline traffic data (default)  |
| `offline` | Uses only offline traffic data, including user-defined roadblocks |

To allow only offline usage:
```dart
TrafficService.preferences.useTraffic = TrafficUsage.offline;
```

---

## Add temporary roadblock during navigation

You can add a roadblock to bypass a portion of the route for a specified distance. Once applied, the route will be recalculated, and the updated route will be returned via the `onRouteUpdated` callback provided to either the `startNavigation` or `startSimulation` method.

To add a 100-meter roadblock starting in 10 meters:
```dart
NavigationService.setNavigationRoadBlock(100, startDistance: 10);
```

Roadblocks added through `setNavigationRoadBlock` only affect the ongoing navigation.

---

## Check traffic information availability

Use the `getOnlineServiceRestrictions` method to check if traffic events are available for a geographic position. It takes a `Coordinates` object as argument and returns a `TrafficOnlineRestrictions` enum.
```dart
Coordinates coords = Coordinates.fromLatLong(50.108, 8.783);
TrafficOnlineRestrictions restriction = TrafficService.getOnlineServiceRestrictions(coords);
```

The `TrafficOnlineRestrictions` enum provides the following values:

- `none` - No restrictions are in place; online traffic is available

- `settings` - Online traffic is disabled in the `TrafficPreferences` object

- `connection` - No internet connection is available

- `networkType` - Not allowed on extra charged networks (e.g., roaming)

- `providerData` - Required provider data is missing

- `worldMapVersion` - The world map version is outdated and incompatible. Please update the road map

- `diskSpace` - Insufficient disk space to download or store traffic data

- `initFail` - Failed to initialize the traffic service

---

## Add persistent roadblock

To add a persistent roadblock, provide the following:

- **startTime** - Timestamp when the roadblock becomes active

- **expireTime** - Timestamp when the roadblock is no longer in effect

- **transportMode** - The specific mode of transport affected by the roadblock

- **id** - A unique string ID for the roadblock

- **coords/area** - Either:

  - A list of coordinates (for path-based roadblocks), or

  - A geographic area (for area-based roadblocks)

The roadblock will affect routing and navigation between the specified **startTime** and **expireTime**. Once the **expireTime** is reached, the roadblock is automatically removed.

The following conditions apply when adding a roadblock:

- If roadblocks are disabled in the `TrafficPreferences` object, the addition will fail with the `GemError.activation` code

- If a roadblock already exists at the same location, the operation will fail with the `GemError.exist` code

- If the input parameters are invalid (e.g., **expireTime** is earlier than **startTime**, missing **id**, or invalid coordinates/area object), the addition will fail with the `GemError.invalidInput` code

- If a roadblock with the same **id** already exists, the addition will fail with the `GemError.inUse` code

### Add area-based persistent roadblock

Use the `addPersistentRoadblockByArea` method to add **area-based** user roadblocks. It accepts a `GeographicArea` object representing the area to be avoided.

The method returns:

- If successful, the newly created `TrafficEvent` instance along with the `GemError.success` code

- If failed, `null` and an appropriate `GemError` code indicating the reason for failure

To add an area-based roadblock starting now and available for 1 hour that affects cars:
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

### Add path-based persistent roadblock

Use the `addPersistentRoadblockByCoordinates` method to add **path-based** user roadblocks. It accepts a list of `Coordinate` objects and supports two modes:

- **Single Coordinate** - Defines a **point-based** roadblock. This may result in two roadblocks being created—one for each travel direction

- **Multiple Coordinates** - Defines a **path-based** roadblock, starting at the first coordinate and ending at the last. This restricts access along a specific road segment

To add a path-based roadblock on both sides of the matching road, starting now and available for 1 hour that affects cars:
```dart
final coords = [Coordinates.fromLatLong(45.64695, 25.62070)];

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

The `addPersistentRoadblockByCoordinates` method may also fail in the following cases:

- **No Suitable Road Found** - If a valid road cannot be identified at the specified coordinates, or if no road data (online or offline) is available for the given location, the method will return `null` along with the `GemError.notFound` error code

- **Route Computation Failed** - If multiple coordinates are provided but a valid route cannot be computed between them, the method will return `null` and the `GemError.noRoute` error code

### Add anti-area persistent roadblock

If a region contains a persistent roadblock, you can whitelist a specific sub-area within the larger restricted zone to allow routing and navigation through that portion. Use the `addAntiPersistentRoadblockByArea` method, which accepts the same arguments as the `addPersistentRoadblockByArea` method.

This enables fine-grained control over blocked regions by allowing exceptions within otherwise restricted areas.

---

## Get all persistent roadblocks

Use the `persistentRoadblocks` getter to retrieve the list of persistent roadblocks. All user-defined roadblocks that are currently active or scheduled to become active are returned. Expired roadblocks are automatically removed.

To iterate through all persistent roadblocks and print their unique identifiers:
```dart
List<TrafficEvent> roadblocks = TrafficService.persistentRoadblocks;

for (final roadblock in roadblocks){
    print(roadblock.description);
}
```

---

## Get persistent roadblock by ID

Use the `getPersistentRoadblock` method to retrieve both path-based and area-based roadblocks by identifier. This method takes the identifier string as argument and returns `null` if the event could not be found or the event if it exists.
```dart
TrafficEvent? event = TrafficService.getPersistentRoadblock("unique_id");

if (event != null){
    print("Event was found");
} else {
    print("Event does not exist");
}
```

---

## Remove roadblocks

### Remove persistent roadblock by ID

Use the `removePersistentRoadblockById` method to remove a roadblock by identifier. This method works for both path-based and area-based roadblocks.
```dart
GemError error = TrafficService.removePersistentRoadblockById("identifier");
if (error == GemError.success){
    print("Removal succeeded");
} else {
    print("Removal failed with error code $error");
}
```

The method returns `GemError.success` if the roadblock was removed and `GemError.notFound` if no roadblock was found with the given ID.

### Remove persistent roadblock by coordinates

Use the `removePersistentRoadblockByCoordinates` method to remove path-based roadblocks by providing the *first* coordinate of the roadblock to be removed.
```dart
GemError error = TrafficService.removePersistentRoadblockByCoordinates(coords);
if (error == GemError.success){
    print("Removal succeeded");
} else {
    print("Removal failed with error code $error");
}
```

The method returns `GemError.success` if the roadblock was removed and `GemError.notFound` if no roadblock was found starting with the given coordinate.

### Remove roadblock using TrafficEvent

If the `TrafficEvent` instance is available, remove it using the `removeUserRoadblock` method:
```dart
TrafficEvent event = ...
TrafficService.removeUserRoadblock(event);
```

This method can be used for both persistent and non-persistent roadblocks.

### Remove all persistent roadblocks

Use the `removeAllPersistentRoadblocks` method to delete all existing user-defined roadblocks.

---

## Get path preview for roadblock

Before adding a persistent roadblock, you can preview the path using an intermediary list of coordinates generated between two positions. The `getPersistentRoadblockPathPreview` method helps visualize the intended roadblock on the map.

This method takes the following arguments:

- `UserRoadblockPathPreviewCoordinate from` - The starting point of the roadblock. Can be constructed from a `Coordinates` object using the `.fromCoordinates()` factory constructor, or returned by the `getPersistentRoadblockPathPreview` method to allow daisy-chaining multiple segments

- `Coordinates to` - The ending point of the roadblock

- `RouteTransportMode transportMode` - The transport mode (e.g., car, bicycle, pedestrian) to be used for the roadblock preview and calculation

The method returns a tuple containing:

- `List<Coordinates>` - A list of intermediate coordinates forming the preview path. This list can be used to render a polyline or marker path on the map

- `UserRoadblockPathPreviewCoordinate` - The updated "end" coordinate, which can be reused as the `from` argument to preview or chain additional segments

- `GemError` - Error code of the operation. This may include the same error codes returned by `addPersistentRoadblockByCoordinates`. The rest of the return values are not valid if the error is not success

To get the preview of a path-based roadblock between two `Coordinate` objects:
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

---

## Listen for roadblock events

You can register for notifications related to persistent roadblocks. These notifications are triggered when:

- A roadblock's `startTime` becomes greater than the current time - via the `onRoadblocksActivated` callback

- A roadblock's `endTime` becomes less than the current time - via the `onRoadblocksExpired` callback

These callbacks provide the activated/expired `List<TrafficEvent>`.

To instantiate a `PersistentRoadblockListener`:
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

The `onRoadblocksActivated` and `onRoadblocksExpired` callbacks can also be registered or changed via the `registerOnRoadblocksExpired` and `registerOnRoadblocksActivated` methods.

Register the listener using the `persistentRoadblockListener` setter:
```dart
TrafficService.persistentRoadblockListener = listener;
```

---

## Relevant examples demonstrating roadblocks related features

- [Draw roadblocks](/examples/maps-3dscene/draw-roadblock)
