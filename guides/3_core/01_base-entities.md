---
description: Documentation for Base Entities
title: Base Entities
---

# Base entities

On this page, we present the simpler ones (coordinates, position, path, geographic areas), while in the following pages we cover the more complex ones (landmarks, markers, overlays, routes).

Reading this helps you understand and use the SDK effectively.

## Coordinates

The `Coordinates` class is a core component designed to represent geographic positions with an optional altitude. The Maps SDK for Flutter uses the [WGS](https://en.wikipedia.org/wiki/World_Geodetic_System) coordinates standard.  Below is an overview of its functionality:

Key Features:

- **Latitude**: Specifies the north-south position. Range: -90.0 to +90.0.

- **Longitude**: Specifies the east-west position. Range: -180.0 to +180.0.

- **Altitude** (optional): Specifies the height in meters. Can be positive or negative.

### Instantiate Coordinates

To create a `Coordinates` instance using latitude and longitude:
```dart
final coordinates = Coordinates(latitude: 48.858844, longitude: 2.294351); // Eiffel Tower
```

Alternatively, the `fromLatLong` constructor can be used:
```dart
final coordinates = Coordinates.fromLatLong(48.858844, 2.294351);
```

### Distance between coordinates

To calculate the distance between two coordinates the `distance` method can be used. This method provides the distance between two coordinates in meters. It also takes into account the altitude if both coordinates have a value for this field.

This method computes the Haversine distance, the shortest path over the Earth’s surface, and returns the result in meters.
```dart
final coordinates1 = Coordinates(latitude: 48.858844, longitude: 2.294351);
final coordinates2 = Coordinates(latitude: 48.854520, longitude: 2.299751);

double distance = coordinates1.distance(coordinates2);
```

The result represents the great-circle distance between the two geographic points and is different from the route distance that would be traveled along roads.

### Copy with meters offset

A new coordinates can be created from an existing one by applying a meter offset. In the example below, the `coordinates2` object is based from the original coordinate, with a 5-meter shift to the north and a 3-meter shift to the east.
```dart
final coordinates1 = Coordinates(latitude: 48.858844, longitude: 2.294351);
final coordinates2 = coordinates1.copyWithMetersOffset(metersLatitude: 5, metersLongitude: 3);
```

The `copyWithMetersOffset` and `distance` methods may exhibit slight inaccuracies.

Coordinates should not be compared using direct equality checks, as minor variations in floating-point precision can lead to inconsistent results.

For example, a coordinate specified as **(48.858395, 2.294469)** may differ slightly from a stored value such as **(48.858394583109785, 2.294469162581987)** due to rounding or internal representation differences. These discrepancies are inherent to floating-point arithmetic and do not indicate a meaningful positional difference.

To ensure reliable comparisons, coordinates should be evaluated using a small numerical tolerance (epsilon) rather than strict equality, preventing false negatives when identifying identical or near-identical geographic locations.

## Path

A `Path` represents a sequence of connected coordinates.

The `Path` class is a core component for representing and managing paths on a map. It offers functionality for path creation, manipulation, and data export, allowing users to define paths and perform various operations programmatically.

Key Features

- **Path Creation & Management**

    - Paths can be created from data buffers in multiple formats (e.g., GPX, KML, GeoJSON).

    - Supports cloning paths in reverse order or between specific coordinates.

- **Coordinates Handling**

    - Provides read-only access to internal coordinates lists.

    - Retrieves a coordinates based on a percentage along the path.

- **Path Properties**

    - **name**: Manage the name of the path.

    - **area**: Retrieve the bounding rectangle of the path.

    - **wayPoints**: Access waypoints along the path.

- **Export Functionality**

    - Export path data in various formats such as GPX, KML, and GeoJSON.

To create a `Path` using coordinates:
```dart
final coords = [
    Coordinates(latitude: 40.786, longitude: -74.202),
    Coordinates(latitude: 40.690, longitude: -74.209),
    Coordinates(latitude: 40.695, longitude: -73.814),
    Coordinates(latitude: 40.782, longitude: -73.710),
];

Path gemPath = Path.fromCoordinates(coords);
```

To create a `Path` from GPX data:
```dart
Uint8List data = ...; // Path data in GPX format
Path path = Path.create(data: data, format: PathFileFormat.gpx);
```

To export a `Path` to some given format (like GeoJson for example) you can proceed like this:
```dart
Uint8List exportedData = path.exportAs(PathFileFormat.geoJson);
```

### Export a Path as String

The `exportAs` method allows you to export a Path into a textual representation. The returned value is a `String` containing the full Path data in the requested format.  
This makes it easy to store the Path as a file or share it with other applications that support formats like GPX, KML, NMEA, or GeoJSON.
```dart
final dataGpx = path.exportAs(PathFileFormat.gpx);
// You now have the full GPX as a string
```

## Geographic areas

Geographic areas represent specific regions of the world and serve various purposes, such as centering, restricting searches to a specific region, geofencing, and more. Multiple entities can return a bounding box as a geographic area, defining the zone that contains the item.

The geographic area types are:

- **Rectangle Geographic Area**: Represents a rectangular area with the top and bottom sides parallel to the longitude and latitude lines.

- **Circle Geographic Area**: Encompasses an area around a specific coordinates with a certain distance.

- **Polygon Geographic Area**: Represents a complex area with high precision, ideal for more detailed geographic boundaries.

At the foundation of the geographic area hierarchy is the abstract `GeographicArea` class, which defines the following operations:

<table>
  <thead>
    <tr>
      <th>Method / Field</th>
      <th>Description</th>
      <th>Return Type</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>boundingBox</code></td>
      <td>Get the bounding box of the geographic area, which is the smallest rectangle surrounding the area.</td>
      <td><code>RectangleGeographicArea</code></td>
    </tr>
    <tr>
      <td><code>convert</code></td>
      <td>Converts the geographic area to another type, if possible.</td>
      <td><code>GeographicArea?</code></td>
    </tr>
    <tr>
      <td><code>centerPoint</code></td>
      <td>Retrieves the center point of the geographic area, calculated as the geographic center.</td>
      <td><code>Coordinates</code></td>
    </tr>
    <tr>
      <td><code>containsCoordinates</code></td>
      <td>Checks if the specified point is contained within the geographic area.</td>
      <td><code>bool</code></td>
    </tr>
    <tr>
      <td><code>isDefault</code></td>
      <td>Checks if the geographic area has default values.</td>
      <td><code>bool</code></td>
    </tr>
    <tr>
      <td><code>type</code></td>
      <td>Retrieves the specific type of the geographic area.</td>
      <td><code>GeographicAreaType</code></td>
    </tr>
      <tr>
      <td><code>reset</code></td>
      <td>Resets the geographic area to its default state.</td>
      <td><code>void</code></td>
    </tr>
  </tbody>
</table>

### Rectangle geographic area

The `RectangleGeographicArea` class represents a rectangular geographic area defined by two coordinates: the top-left and bottom-right corners. It provides operations to check for intersections, containment, and unions with other rectangles.

To create a new `RectangleGeographicArea`, the constructor can be used by providing the top-left and bottom-right coordinates.
```dart
final topLeftCoords = Coordinates(latitude: 44.93343, longitude: 25.09946);
final bottomRightCoords = Coordinates(latitude: 44.93324, longitude: 25.09987);
final area = RectangleGeographicArea(topLeft: topLeftCoords, bottomRight: bottomRightCoords);
```

A valid ``RectangleGeographicArea`` should have the latitude of `topLeft` coordinates greater than the latitude of the `bottomRight` coordinates and the longitude of `topLeft` coordinates smaller than the longitude of `bottomRight` coordinate.

### Circle geographic area

The `CircleGeographicArea` class represents a circular geographic area defined by a center point and a radius. It provides methods for checking if a point lies within the circle, calculating the bounding box, and more.

To create a new `CircleGeographicArea`, the constructor can be used by providing the center point and the distance in meters:
```dart
final center = Coordinates(latitude: 40.748817, longitude: -73.985428);

final circle = CircleGeographicArea(
  radius: 500, 
  centerCoordinates: center,
);
```

### Polygon geographic area

The `PolygonGeographicArea` class can be used to represent complex custom areas with a high level of precision.

They can be created by providing the list of coordinates:
```dart
List<Coordinates> coordinates = [
    Coordinates(latitude: 10, longitude: 0),
    Coordinates(latitude: 10, longitude: 10),
    Coordinates(latitude: 0, longitude: 10),
    Coordinates(latitude: 0, longitude: 0),
];

PolygonGeographicArea polygonGeographicArea = PolygonGeographicArea(coordinates: coordinates);
```

A valid `PolygonGeographicArea` should have at least 3 coordinates. Avoid overlapping and intersecting edges.
