---
description: Documentation for Public Transit Stops
title: Public Transit Stops
---

# Public Transit stops

This API provides detailed access to public transport data including agencies, routes, stops, and trips. It is designed to integrate with interactive map-based applications using the Magic Lane SDK and allows developers to dynamically fetch and explore real-time public transportation information from selected positions on the map.

Key Features

- Query public transport overlays by screen position.

- Retrieve structured information about transport agencies, stops, routes, and trips.

- Support for real-time data including delays and cancellations.

- Metadata about accessibility, bike allowances, and platform details.

- Utilities for filtering trips by route type, route short name, or agency.

How It Works

- Set a position on the map using `setCursorScreenPosition`.

- Query for public transport overlays with `cursorSelectionOverlayItemsByType` or for generic overlays with `cursorSelectionOverlayItems`.

- Retrieve stop information using `getPTStopInfo()` on each overlay item.

- Use the returned `PTStopInfo` object to explore agencies, stops, and trips.

- This API is part of a modular system intended to support rich and interactive transit applications.

An example of using this feature is the following:
```dart
void _onMapCreated(GemMapController controller) async {
    // Save controller for further usage.
    _mapController = controller;

    _mapController.registerLongPressCallback((pos) async {
        // set cursor position on the screen
        await _mapController.setCursorScreenPosition(pos);

        // get the public transit overlay items at that position
        final items = _mapController
            .cursorSelectionOverlayItemsByType(CommonOverlayId.publicTransport);

        // for each overlay item at that position
        for (final OverlayItem item in items) {
            // get the stop information
            final ptStopInfo = await item.getPTStopInfo();
            if (ptStopInfo != null) {
                // information about agencies
                final agencies = ptStopInfo.agencies;

                // information about stops and generic routes
                // (routes that don't have `heading` set)
                final stops = ptStopInfo.stops;

                // information about trips (together with 
                // route, agency, stop times, real time info, etc.)
                final trips = ptStopInfo.trips;

                // How to use stops
                for (final stop in stops) {
                    print('Stop id: ${stop.stopId}');
                    print('Stop name: ${stop.stopName}');

                    print('Routes:');
                    for (final route in stop.routes) {
                        print('  Route id: ${route.routeId}');
                        print('  Route short name: ${route.routeShortName}');
                        print('  Route long name: ${route.routeLongName}');
                    }
                }
            }
        }
    });
}
```

You can also obtain instances of `PTStopInfo` by performing an overlay search using `CommonOverlayId.publicTransport`. Once you retrieve the corresponding `OverlayItem`s, use their `getPTStopInfo` method to access the stop information.

See the [Search on overlays](./search/get-started-search#search-on-overlays) geuide for more details.

All returned times are local times! This means they are created as `DateTime`(s) with times as UTC (timezone offset is 0).

Use the `TimezoneService` in order to make conversions.

There are two types of public transit stops on the map:

- Stops which are of type `OverlayItem` and can be selected via the `cursorSelectionOverlayItemsByType` method. These stops provide extensive details structured as `PTStopInfo` objects. They are displayed with a blue icon when using the default style.

- Stops which are of type `Landmark` and can be selected via the `cursorSelectionLandmarks` method. These stops do not provide extensive details. They are displayed with a gray icon when using the default style.

You can filter the trips by:

- route short name (`tripsByRouteShortName`)

- route type(`tripsByRouteType`)

- agency (`tripsByAgency`)

These can be accomplished by using the following methods of the `PTStopInfo` class:
```dart
List<PTTrip> tripsByRouteShortName(String name)
List<PTTrip> tripsByRouteType(PTRouteType type)
List<PTTrip> tripsByAgency(PTAgency agency)
```

An example illustrating this is the following:
```dart
final trips = ptStopInfo.tripsByRouteType(PTRouteType.bus);
```

## PTAgency

Represents a public transport agency.

| Property | Type | Description |
|----------|------|-------------|
| `id` | `int` | Agency ID |
| `name` | `String` | Agency name |
| `url` | `String?` | Optional website URL |

---

## PTRouteType

Enum representing route types.
```dart
enum PTRouteType {
  bus,
  underground,
  railway,
  tram,
  waterTransport,
  misc
}
```

---

## PTRouteInfo

Represents a public transport route.

| Property | Type | Description |
|----------|------|-------------|
| `routeId` | `int` | Route ID |
| `routeShortName` | `String?` | Short name |
| `routeLongName` | `String?` | Long name |
| `routeType` | `PTRouteType` | Type of route |
| `routeColor` | `Color?` | Color in hex |
| `routeTextColor` | `Color?` | Text color in hex |
| `heading` | `String?` | Optional heading |

---

Do not confuse the `PTRoute` and `PTRouteInfo` classes.

- `PTRouteInfo` provides information about the public transit routes available at a specific stop.

- `PTRoute`, on the other hand, represents a computed public transit route between multiple waypoint and includes detailed instructions.

For more information on computing public transit routes using `PTRoute`, refer to the [Compute Public Transit Routes](/guides/routing/advanced-features#compute-public-transit-routes) section.

## PTStop

Represents a public transport stop.

| Property | Type | Description |
|----------|------|-------------|
| `stopId` | `int` | Stop ID |
| `stopName` | `String` | Stop name |
| `isStation` | `bool?` | Is station or not |
| `routes` | `List<PTRouteInfo>` | Associated routes |

---

## PTStopTime

Details a stop time in a trip.

| Property | Type | Description |
|----------|------|-------------|
| `stopName` | `String` | Stop name |
| `coordinates` | `Coordinates` | Lat/Lon |
| `hasRealtime` | `bool` | Real-time available |
| `delay` | `int` | Delay in seconds |
| `departureTime` | `DateTime?` | Optional departure time |
| `isBefore` | `bool` | Is before current time |
| `isWheelchairFriendly` | `bool` | Wheelchair accessible |

---

## PTTrip

Describes a public transport trip.

| Property | Type | Description |
|----------|------|-------------|
| `route` | `PTRouteInfo` | Associated route |
| `agency` | `PTAgency` | Associated agency |
| `tripIndex` | `int` | Trip index |
| `tripDate` | `DateTime?` | Date |
| `departureTime` | `DateTime?` | Departure |
| `hasRealtime` | `bool` | Realtime info |
| `isCancelled` | `bool?` | Cancelled |
| `delayMinutes` | `int?` | Delay |
| `stopTimes` | `List<PTStopTime>` | Times at stops |
| `stopIndex` | `int` | Stop index |
| `stopPlatformCode` | `String` | Platform |
| `isWheelchairAccessible` | `bool` | Wheelchair access |
| `isBikeAllowed` | `bool` | Bike allowed |

---

## PTStopInfo

Aggregates stop-related data.

| Property | Type | Description |
|----------|------|-------------|
| `agencies` | `List<PTAgency>` | Available agencies |
| `stops` | `List<PTStop>` | Nearby stops |
| `trips` | `List<PTTrip>` | Available trips |
