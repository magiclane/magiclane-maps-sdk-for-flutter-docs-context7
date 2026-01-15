---
description: Documentation for Traffic Events
title: Traffic Events
---

# Traffic Events

The Maps SDK for Flutter provides real-time information about traffic events such as delays, roadworks, and accidents.

When enabled and supported by your map style, traffic events appear as red overlays on affected road segments.

**Event sources:**

- **Magic Lane servers** - Provide up-to-date traffic data when online

- **User-defined roadblocks** - Blacklist specific road segments or areas

**Impact zones:**

- **Path-based** - Follows the shape of a road

- **Area-based** - Covers a larger geographic area

The central class for handling traffic events and roadblocks is `TrafficEvent`. You can obtain instances through user interaction with the map or as part of roadblock operations. For route-specific traffic data, use the `RouteTrafficEvent` class, which extends `TrafficEvent` with detailed route information.

Traffic events, including delays and user-defined roadblocks, are fully integrated into the routing and navigation logic. This ensures that calculated routes dynamically account for traffic conditions and any restricted segments.

## TrafficEvent structure

The `TrafficEvent` class contains the following members:

| Member                     | Type                         | Description |
|----------------------------|------------------------------|-------------|
| `isRoadblock`              | `bool`                       | Returns `true` if the event represents a roadblock. |
| `delay`                    | `int`                        | Estimated delay in seconds. Returns `-1` if unknown. |
| `length`                   | `int`                        | Length in meters of the road segment affected. Returns `-1` if unknown if the event is an area-based roadblock. |
| `impactZone`               | `TrafficEventImpactZone`     | Indicates if the event affects a point or area. |
| `referencePoint`           | `Coordinates`                | The central coordinate for the event. Returns `(0,0)` for area events. |
| `boundingBox`              | `RectangleGeographicArea`    | Geographical bounding box surrounding the event. |
| `area`                     | `GeographicArea`             | The geographic area associated with the event (see `TrafficService.addPersistentRoadblockByArea`). If no area is provided, this is the same as `boundingBox`. |
| `isAntiArea`               | `bool`                       | Check if the impact zone is the anti-area of the event area. Valid only for area impact zone. |
| `isActive`                 | `bool`                       | Check if traffic event is active ( i.e. is started ). |
| `isExpired`                | `bool`                       | Check if traffic event is expired ( i.e. is ended ). |
| `description`              | `String`                     | Human-readable description of the traffic event. If it is a user-defined roadblock contains the id |
| `eventClass`               | `TrafficEventClass`          | Classification of the traffic event. |
| `eventSeverity`            | `TrafficEventSeverity`       | Severity level of the event. |
| `getImage({size, format})` | `Uint8List?`                 | Returns an image representing the traffic event. |
| `img`                      | `Img`                        | Retrieves the event image in internal format (`Img`). |
| `previewUrl`               | `String`                     | Returns a URL to preview the traffic event. Returns emtpy if not available (user-defined roadblock). |
| `isUserRoadblock`          | `bool`                       | Returns `true` if the event is a user-defined roadblock. |
| `affectedTransportModes`   | `Set<TrafficTransportMode>`  | Returns all transport modes affected by the event. |
| `startTime`                | `DateTime?`                  | UTC start time of the traffic event, if available. |
| `endTime`                  | `DateTime?`                  | UTC end time of the traffic event, if available. |
| `hasOppositeSibling`       | `bool`                       | Returns `true` if a sibling event exists in the opposite direction. Relevant for path-based events. |

---

## Understand RouteTrafficEvent structure

The `RouteTrafficEvent` class extends `TrafficEvent` with route-specific information:

| Member                      | Type                                | Description |
|-----------------------------|-------------------------------------|-------------|
| `distanceToDestination`     | `int`                               | Returns the distance in meters from the event's position on the route to the destination. Returns 0 if unavailable. |
| `from`                      | `Coordinates`                       | The starting point of the traffic event on the route. |
| `to`                        | `Coordinates`                       | The end point of the traffic event on the route. |
| `fromLandmark`              | `(Landmark, bool)`                  | Returns the starting point as a landmark and a flag indicating if data is cached locally. |
| `toLandmark`                | `(Landmark, bool)>`                 | Returns the end point as a landmark and a flag indicating if data is cached locally. |
| `asyncUpdateToFromData()`   | `void Function(GemError)`           | Asynchronously updates the `from` and `to` landmarks' address and description info from the server. |
| `cancelUpdate()`            | `void`                              | Cancels the pending async update request for landmark data. |

---

## Use traffic events

Traffic events provide insights into road conditions, delays, and closures:

- **Route traffic information** - See the [Get ETA and Traffic information guide](../routing/get-started-routing#retrieve-time-and-distance-information)

- **User-defined roadblocks** - See the [Roadblocks guide](../navigation/roadblocks)

### Event classifications

The `TrafficEventClass` provides the following event types:

- `trafficRestrictions`

- `roadworks`

- `parking`

- `delays`

- `accidents`

- `roadConditions`

### Severity levels

The `TrafficEventSeverity` enum includes:

- `stationary`

- `queuing`

- `slowTraffic`

- `possibleDelay`

- `unknown`
