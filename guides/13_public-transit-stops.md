---
description: Documentation for Public Transit Stops
title: Public Transit Stops
---

# Public Transit stops

This API provides detailed access to public transport data including agencies, routes, stops, and trips. It is designed to integrate with interactive map-based applications using the Magic Lane SDK and allows developers to dynamically fetch and explore real-time public transportation information from selected positions on the map.

The structure of the public transport data is modeled after the [General Transit Feed Specification (GTFS)](https://gtfs.org/documentation/schedule/reference/) and offers access to a subset of the fields and entities defined in GTFS.

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

    _mapController.registerOnLongPress((pos) async {
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
                // route, agency, stop times, real-time info, etc.)
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

See the [Search on overlays](./search/get-started-search#search-on-overlays) guide for more details.

All returned times are local times. They are represented as `DateTime` values in UTC (timezone offset 0).

Use the `TimezoneService` to convert them to other time zones.

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

## Agencies

The `PTAgency` class represents a public transport agency.

| Property | Type | Description |
|----------|------|-------------|
| `id` | `int` | Agency ID |
| `name` | `String` | Full name of the transit agency. |
| `url` | `String?` | Optional URL of the transit agency. |

## Public Transport Routes

The `PTRouteInfo` class represents a public transport route.

| Property | Type | Description |
|----------|------|-------------|
| `routeId` | `int` | Route ID |
| `routeShortName` | `String?` | Short name of a route. Often a short, abstract identifier (e.g., "32", "100X") that riders use to identify a route. May be null. |
| `routeLongName` | `String?` | Full name of a route. This name is generally more descriptive than the short name and often includes the route's destination or stop. |
| `routeType` | `PTRouteType` | Type of route.|
| `routeColor` | `Color?` | Route color designation that matches public-facing material. May be used to color the route on the map or to be shown on UI elements. |
| `routeTextColor` | `Color?` | Legible color to use for text drawn against a background of `routeColor`. |
| `heading` | `String?` | Optional heading information. |

Do not confuse the `PTRoute` and `PTRouteInfo` classes.

- `PTRouteInfo` provides information about the public transit routes available at a specific stop.

- `PTRoute`, on the other hand, represents a computed public transit route between multiple waypoints and includes detailed instructions.

For more information on computing public transit routes using `PTRoute`, refer to the [Compute Public Transit Routes](/guides/routing/advanced-features#compute-public-transit-routes) section.

The type of public transport route is represented by the `PTRouteType` enum.

| Enum Case       | Description                                                                                   |
|-----------------|-----------------------------------------------------------------------------------------------|
| `bus`           | Bus, Trolleybus. Used for short and long-distance bus routes.                                 |
| `underground`   | Subway, Metro. Any underground rail system within a metropolitan area.                        |
| `railway`       | Rail. Used for intercity or long-distance travel.                                             |
| `tram`          | Tram, Streetcar, Light rail. Any light rail or street level system within a metropolitan area.|
| `waterTransport`| Water transport. Used for ferries and other water-based transit.                              |
| `misc`          | Miscellaneous. Includes other types of public transport not covered by the other categories.  |

## Stops

The `PTStop` class represents a public transport stop.

| Property | Type | Description |
|----------|------|-------------|
| `stopId` | `int` | Identifies a location: stop/platform, station, entrance/exit, node or boarding area  |
| `stopName` | `String` | Name of the location. Matches the agency's rider-facing name for the location as printed on a timetable, published online, or represented on signage. |
| `isStation` | `bool?` | Whether this location is a station or not. A station is considered a physical structure or area that contains one or more platforms. |
| `routes` | `List<PTRouteInfo>` | Associated routes for the stop. Contains all routes serving this stop, whether active at the given time or not. |

## Stop Times

The `PTStopTime` class provides details about stop time in a `PTTrip`.

| Property | Type | Description |
|----------|------|-------------|
| `stopName` | `String` | The name of the serviced stop. |
| `coordinates` | `Coordinates` | WGS latitude and longitude for the stop. |
| `hasRealtime` | `bool` | Whether data is provided in real-time or not. |
| `delay` | `int` | Delay in seconds. Not available if `hasRealtime` is false. |
| `departureTime` | `DateTime?` | Optional departure time in the local timezone. |
| `isBefore` | `bool` | Whether the stop time is before the current time. |
| `isWheelchairFriendly` | `bool` | Whether the stop is wheelchair accessible. |

## Trips

The `PTTrip` class represents a public transport trip.

| Property | Type | Description |
|----------|------|-------------|
| `route` | `PTRouteInfo` | Associated route |
| `agency` | `PTAgency` | Associated agency |
| `tripIndex` | `int` | Trip index |
| `tripDate` | `DateTime?` | The date of the trip |
| `departureTime` | `DateTime?` | Departure time of the trip from the first stop |
| `hasRealtime` | `bool` | Whether real-time data is available |
| `isCancelled` | `bool?` | Whether the trip is cancelled |
| `delayMinutes` | `int?` | Delay in minutes. Not available if `hasRealtime` is false. |
| `stopTimes` | `List<PTStopTime>` | Details of stop times in the trip |
| `stopIndex` | `int` | Stop index |
| `stopPlatformCode` | `String` | Platform code |
| `isWheelchairAccessible` | `bool` | Whether the stop is wheelchair accessible. |
| `isBikeAllowed` | `bool` | Whether bikes are allowed on the stop. |

Do not confuse the `PTRoute` and `PTTrip` classes.

The `PTRouteInfo` class represents the public-facing service that riders recognize, like “Bus 42” while a `PTTrip` is a single scheduled journey along that route at a specific time, with its own stop times and sequence.

In other words, the route is the line or service identity, and the trips are the individual vehicle runs that make up that service throughout the day.

## Stop Info

The `PTStopInfo` class aggregates stop-related data including agencies, stops, and trips related to a specific public transit overlay item.

| Property    | Type             | Description |
|-------------|------------------|-------------|
| `agencies`  | `List<PTAgency>` | Agencies serving the selected item |
| `trips`     | `List<PTTrip>`   | Trips in which the selected item is involved |
| `stops`     | `List<PTStop>`   | Stops associated with the trips |
