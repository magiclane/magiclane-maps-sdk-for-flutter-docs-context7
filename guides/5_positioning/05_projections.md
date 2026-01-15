---
description: Documentation for Projections
title: Projections
---

# Projections

The Maps SDK for Flutter provides a `Projection` class that represents the base class for different geocoordinate systems.

---

## Supported projection types

- `WGS84` - World Geodetic System 1984

- `GK` - Gauss-Kruger

- `UTM` - Universal Transverse Mercator

- `LAM` - Lambert

- `BNG` - British National Grid

- `MGRS` - Military Grid Reference System

- `W3W` - What three words

You can check the projection type using the `type` getter:
```dart
final type = projection.type;
```

---

## WGS84 projection

The **WGS84** projection is a widely used geodetic datum that serves as the foundation for GPS and other mapping systems.

Create a `WGS84` projection using a `Coordinates` object:
```dart
final obj = WGS84Projection(Coordinates(latitude: 5.0, longitude: 5.0));
```

Access and modify coordinates using the `coordinates` getter and setter:
```dart
final coordinates = obj.coordinates; // Get coordinates
obj.coordinates = Coordinates(latitude: 10.0, longitude: 10.0); // Set coordinates
```

The coordinates getter returns null if the coordinates are not set.

---

## GK projection

The **Gauss-Kruger** projection is a cylindrical map projection commonly used for large-scale mapping in regions with a north-south orientation. It divides the Earth into zones, each with its own coordinate system.

Create a `Gauss-Kruger` projection:
```dart
final obj = GKProjection(x: 6325113.72, y: 5082540.66, zone: 1);
```

Access values using the `easting`, `northing` and `zone` getters. Modify them using the `setFields` method:
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

The `Gauss-Kruger` projection is currently supported only for countries that use **Bessel ellipsoid**. Converting to and from `Gauss-Kruger` projection for other countries will result in a `GemError.notSupported` error.

---

## BNG projection

The **BNG** (British National Grid) projection is a coordinate system used in Great Britain for mapping and navigation. It provides a grid reference system for precise location identification.

Create a `BNG` projection:
```dart
final obj = BNGProjection(easting: 500000, northing: 4649776);
```

Access values using the `easting` and `northing` getters. Modify them using the `setFields` method:
```dart
final obj = BNGProjection(easting: 6325113.72, northing: 5082540.66);

// highlight-start
obj.setFields(easting: 1, northing: 1);
// highlight-end

final type = obj.type; // ProjectionType.bng
final newEasting = obj.easting; // 1
final newNorthing = obj.northing; // 1
```

---

## MGRS projection

The **MGRS** (Military Grid Reference System) projection is a coordinate system used by the military for precise location identification. It combines the UTM and UPS coordinate systems.

Create a `MGRS` projection:
```dart
final obj = MGRSProjection(easting: 99316, northing: 10163, zone: '30U', letters: 'XC');
```

Access values using the `easting`, `northing`, `zone` and `letters` getters. Modify them using the `setFields` method:
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

---

## W3W projection

The **W3W** (What three words) projection is a geocoding system that divides the world into a grid of 3m x 3m squares, each identified by a unique combination of three words.

Create a `W3W` projection:
```dart
final obj = W3WProjection('token');
```

Access and modify the token and words values using the `token` and `words` getters and setters.

---

## LAM projection

The **LAM** (Lambert) projection is a conic map projection commonly used for large-scale mapping in regions with an east-west orientation.

Create a `LAM` projection:
```dart
final obj = LAMProjection(x: 6325113.72, y: 5082540.66);
```

Access values using the `x` and `y` getters. Modify them using the `setFields` method.
```dart
final obj = LAMProjection(x: 6325113.72, y: 5082540.66);

// highlight-start
obj.setFields(x: 1, y: 1);
// highlight-end

final type = obj.type; // ProjectionType.lam
final newX = obj.x; // 1
final newY = obj.y; // 1
```

---

## UTM projection

The **UTM** (Universal Transverse Mercator) projection is a global map projection that divides the world into a series of zones, each with its own coordinate system.

Create a `UTM` projection:
```dart
final obj = UTMProjection(x: 6325113.72, y: 5082540.66, zone: 1, hemisphere: Hemisphere.south);
```

Access values using the `x`, `y`, `zone` and `hemisphere` getters. Modify them using the `setFields` method:
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

---

## Convert between projections

The `ProjectionService` class provides a method to convert between different projection types. Use the static `convert` method to transform coordinates from one projection to another:
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

`ProjectionService.convert` works with `W3WProjection` only if the `W3WProjection` object has a **valid** token that can be obtained from [what3words.com](https://developer.what3words.com/public-api). If the token is not set, the conversion will fail and the `GemError.notSupported` error will be returned via `onComplete`.

---

## Relevant examples demonstrating projections related features

- [Projections](/examples/maps-3dscene/projections)
