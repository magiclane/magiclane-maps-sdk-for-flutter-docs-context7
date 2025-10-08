---
description: Documentation for Projections
title: Projections
---

# Projections

Besides the `Coordinates` class, the Maps SDK for Flutter provides a `Projection` class that represents the base class for different geocoordinate systems such as:

- `WGS84` (World Geodetic System 1984)

- `GK` (Gauss-Kruger)

- `UTM` (Universal Transverse Mercator)

- `LAM` (Lambert)

- `BNG` (British National Grid)

- `MGRS` (Military Grid Reference System)

- `W3W` (What three words)

To know the type of the `Projection` you can use the `type` getter:
```dart
final type = projection.type;
```

## WGS84 Projection

The `WGS84` projection is a widely used geodetic datum that serves as the foundation for GPS and other mapping systems. It provides a standard reference frame for the Earth's surface, allowing for accurate positioning and navigation. A `WGS84` projection can be instantiated using a `Coordinates` object:
```dart
final obj = WGS84Projection(Coordinates(latitude: 5.0, longitude: 5.0));
```

Then, the coordinates can be accessed and set using the `coordinates` getter and setter:
```dart
final coordinates = obj.coordinates; // Get coordinates
obj.coordinates = Coordinates(latitude: 10.0, longitude: 10.0); // Set coordinates
```

The coordinates getter returns null if the coordinates are not set.

## GK Projection

The `Gauss-Kruger` projection is a cylindrical map projection that is commonly used for large-scale mapping in regions with a north-south orientation. It divides the Earth into zones, each with its own coordinate system, allowing for accurate representation of geographic features. A `Gauss-Kruger` projection can be instantiated using the following constructor:
```dart
final obj = GKProjection(x: 6325113.72, y: 5082540.66, zone: 1);
```

In order to obtain the x, y and zone values, the `easting`, `northing` and `zone` getters can be used, while setting them can be done using the `setFields` method:
```dart
final obj = GKProjection(x: 6325113.72, y: 5082540.66, zone: 1);

final type = obj.type; // ProjectionType.gk
final zone = obj.zone; // 1
final easting = obj.easting; // 6325113.72
final northing = obj.northing; // 5082540.66

// highlight-start
obj.setFields(x: 1, y: 1, zone: 2);
// highlight-end

final newZone = obj.zone; // 2
final newEasting = obj.easting; // 1
final newNorthing = obj.northing; // 1
```

The `Gauss-Kruger` projection is currently supported only for countries that use **Bessel ellipsoid**. Trying to convert to and from `Gauss-Kruger` projection for other countries will result in a `GemError.notSupported` error.

## BNG Projection

The `BNG` (British National Grid) projection is a coordinate system used in Great Britain for mapping and navigation. It provides a grid reference system that allows for precise location identification within the country. A `BNG` projection can be instantiated using the following constructor:
```dart
final obj = BNGProjection(easting: 500000, northing: 4649776);
```

In order to obtain the easting and northing values, the `easting` and `northing` getters can be used, while setting them can be done using the `setFields` method:
```dart
final obj = BNGProjection(easting: 6325113.72, northing: 5082540.66);

// highlight-start
obj.setFields(easting: 1, northing: 1);
// highlight-end

final type = obj.type; // ProjectionType.bng
final newEasting = obj.easting; // 1
final newNorthing = obj.northing; // 1
```

## MGRS Projection

The `MGRS` (Military Grid Reference System) projection is a coordinate system used by the military for precise location identification. It combines the UTM and UPS coordinate systems to provide a grid reference system that is easy to use in the field. A `MGRS` projection can be instantiated using the following constructor:
```dart
final obj = MGRSProjection(easting: 99316, northing: 10163, zone: '30U', letters: 'XC');
```

In order to obtain the easting, northing, zone and letters values, the `easting`, `northing`, `zone` and `letters` getters can be used, while setting them can be done using the `setFields` method:
```dart
final obj = MGRSProjection(
    easting: 6325113, northing: 5082540, zone: 'A', letters: 'letters');

// highlight-start
obj.setFields(
    easting: 1, northing: 1, zone: 'B', letters: 'newLetters');
// highlight-end

final type = obj.type; // ProjectionType.mgrs
final newZone = obj.zone; // B
final newEasting = obj.easting; // 1
final newNorthing = obj.northing; // 1
final newLetters = obj.letters; // newLetters
```

## W3W Projection

The `W3W` (What three words) projection is a geocoding system that divides the world into a grid of 3m x 3m squares, each identified by a unique combination of three words. This system provides a simple and memorable way to reference specific locations. A `W3W` projection can be instantiated using the following constructor:
```dart
final obj = W3WProjection('token');
```

In order to obtain and set the token and words values, the `token` and `words` getters and setters can be used.

## LAM Projection

The `LAM` (Lambert) projection is a conic map projection that is commonly used for large-scale mapping in regions with an east-west orientation. It provides a way to represent geographic features accurately while minimizing distortion. A `LAM` projection can be instantiated using the following constructor:
```dart
final obj = LAMProjection(x: 6325113.72, y: 5082540.66);
```

In order to obtain the x and y values, the `x` and `y` getters can be used, while setting them can be done using the `setFields` method.
```dart
final obj = LAMProjection(x: 6325113.72, y: 5082540.66);

// highlight-start
obj.setFields(x: 1, y: 1);
// highlight-end

final type = obj.type; // ProjectionType.lam
final newX = obj.x; // 1
final newY = obj.y; // 1
```

## UTM Projection

The `UTM` (Universal Transverse Mercator) projection is a global map projection that divides the world into a series of zones, each with its own coordinate system. It provides a way to represent geographic features accurately while minimizing distortion. A `UTM` projection can be instantiated using the following constructor:
```dart
final obj = UTMProjection(x: 6325113.72, y: 5082540.66, zone: 1, hemisphere: Hemisphere.south);
```

In order to obtain the x, y, zone and hemisphere values, the `x`, `y`, `zone` and `hemisphere` getters can be used, while setting them can be done using the `setFields` method:
```dart
final obj = UTMProjection(x: 6325113.72, y: 5082540.66, zone: 1, hemisphere: Hemisphere.south);

// highlight-start
obj.setFields(x: 1, y: 1, zone: 2, hemisphere: Hemisphere.north);
// highlight-end

final type = obj.type; // ProjectionType.utm
final newZone = obj.zone; // 2
final newX = obj.x; // 1
final newY = obj.y; // 1
final newHemisphere = obj.hemisphere; // Hemisphere.north
```

## Projection Service

The `ProjectionService` class provides a method to convert between different projection types. It allows you to transform coordinates from one projection to another, making it easier to work with various geospatial data formats. The class is abstract and features a static `convert` method:
```dart
final from = WGS84Projection(Coordinates(latitude: 51.5074, longitude: -0.1278));
final toType = ProjectionType.mgrs;

final completer = Completer<Projection?>();
ProjectionService.convert(
    from: from,
    toType: toType,
    onComplete: (error, result) {
        if (error == GemError.success) {
            completer.complete(result);
        } else {
            completer.completeError(error);
        }
    },
);

final result = await completer.future;
final mgrs = result as MGRSProjection;

final easting = mgrs.easting; // 99316
final northing = mgrs.northing; // 10163
final zone = mgrs.zone; // 30U
final letters = mgrs.letters; // XC
```

ProjectionService.convert works with `W3WProjection` only if the `W3WProjection` object has a **valid** token that can be obtained from [what3words.com](https://developer.what3words.com/public-api). If the token is not set, the conversion will fail and the `GemError.notSupported` error will be returned via `onComplete`.

## Relevant example demonstrating projections related features

- [Projections](/examples/maps-3dscene/projections)
