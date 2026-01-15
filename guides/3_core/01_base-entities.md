---
description: Documentation for Base Entities
title: Base Entities
---

# Base entities

This page covers the fundamental building blocks of the SDK: coordinates, paths, and geographic areas.

---

## Coordinates

The `Coordinates` class represents geographic positions with an optional altitude. The Maps SDK for Flutter uses the [WGS](https://en.wikipedia.org/wiki/World_Geodetic_System) coordinates standard.

**Key components:**

- **Latitude** - North-south position. Range: -90.0 to +90.0

- **Longitude** - East-west position. Range: -180.0 to +180.0

- **Altitude** - Height in meters (optional). Can be positive or negative

### Create coordinates

Create a `Coordinates` instance using latitude and longitude:
```dart
final coordinates = Coordinates(latitude: 48.858844, longitude: 2.294351); // Eiffel Tower
```

Alternatively, use the `fromLatLong` constructor:
```dart
final coordinates = Coordinates.fromLatLong(48.858844, 2.294351);
```

### Calculate distance

The `distance` method calculates the distance between two coordinates in meters. It accounts for altitude if both coordinates have this value.

This method computes the Haversine distance (the shortest path over the Earth's surface).
```dart
final coordinates1 = Coordinates(latitude: 48.858844, longitude: 2.294351);
final coordinates2 = Coordinates(latitude: 48.854520, longitude: 2.299751);

double distance = coordinates1.distance(coordinates2);
```

The result represents the great-circle distance between the two geographic points and is different from the route distance that would be traveled along roads.

### Apply offset

Create new coordinates from existing ones by applying a meter offset. In this example, `coordinates2` is shifted 5 meters north and 3 meters east from the original:
```dart
final coordinates1 = Coordinates(latitude: 48.858844, longitude: 2.294351);
final coordinates2 = coordinates1.copyWithMetersOffset(metersLatitude: 5, metersLongitude: 3);
```

The `copyWithMetersOffset` and `distance` methods may exhibit slight inaccuracies.

Coordinates should not be compared using direct equality checks, as minor variations in floating-point precision can lead to inconsistent results.

For example, **(48.858395, 2.294469)** may differ slightly from a stored value like **(48.858394583109785, 2.294469162581987)** due to rounding or internal representation differences. These discrepancies are inherent to floating-point arithmetic and do not indicate a meaningful positional difference.

Use a small numerical tolerance (epsilon) for reliable comparisons rather than strict equality.

---

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

### Create from data

Create a `Path` from GPX data:
```dart
Uint8List data = ...; // Path data in GPX format
Path path = Path.create(data: data, format: PathFileFormat.gpx);
```

### Export path data

Export a `Path` to a specific format (like GeoJSON):
```dart
Uint8List exportedData = path.exportAs(PathFileFormat.geoJson);
```

The `exportAs` method returns a `String` containing the full path data in the requested format. This makes it easy to store the path as a file or share it with other applications that support GPX, KML, NMEA, or GeoJSON.
```dart
final dataGpx = path.exportAs(PathFileFormat.gpx);
// You now have the full GPX as a string
```

---

## Geographic areas

Geographic areas represent specific regions for centering, search restrictions, geofencing, and more. Multiple entities can return a bounding box as a geographic area.

**Available types:**

- **Rectangle Geographic Area** - Rectangular area with sides parallel to longitude and latitude lines

- **Circle Geographic Area** - Area around a center point with a specified radius

- **Polygon Geographic Area** - Complex area with high precision for detailed boundaries

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

The `RectangleGeographicArea` class represents a rectangular area defined by top-left and bottom-right corners. It supports operations for intersections, containment, and unions.

Create a `RectangleGeographicArea` by providing the corner coordinates:
```dart
final topLeftCoords = Coordinates(latitude: 44.93343, longitude: 25.09946);
final bottomRightCoords = Coordinates(latitude: 44.93324, longitude: 25.09987);
final area = RectangleGeographicArea(topLeft: topLeftCoords, bottomRight: bottomRightCoords);
```

A valid `RectangleGeographicArea` requires the latitude of `topLeft` to be greater than `bottomRight`, and the longitude of `topLeft` to be smaller than `bottomRight`.

### Circle geographic area

The `CircleGeographicArea` class represents a circular area defined by a center point and radius. It supports containment checks and bounding box calculations.

Create a `CircleGeographicArea` by providing the center point and radius in meters:
```dart
final center = Coordinates(latitude: 40.748817, longitude: -73.985428);

final circle = CircleGeographicArea(
  radius: 500, 
  centerCoordinates: center,
);
```

### Polygon geographic area

The `PolygonGeographicArea` class represents complex custom areas with high precision.

Create a polygon by providing a list of coordinates:
```dart
List<Coordinates> coordinates = [
    Coordinates(latitude: 10, longitude: 0),
    Coordinates(latitude: 10, longitude: 10),
    Coordinates(latitude: 0, longitude: 10),
    Coordinates(latitude: 0, longitude: 0),
];

PolygonGeographicArea polygonGeographicArea = PolygonGeographicArea(coordinates: coordinates);
```

A valid `PolygonGeographicArea` requires at least 3 coordinates. Avoid overlapping and intersecting edges.
