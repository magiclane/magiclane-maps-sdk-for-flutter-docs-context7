---
description: Documentation for Migrate To 3 1 0
title: Migrate To 3 1 0
---

# Migrate to 3.1.0

This guide outlines the breaking changes introduced in SDK version 3.1.0. Required updates may vary depending on your use case.

Additionally, new features and bugfixes have been introduced and are **not** documented here. For a comprehensive list of changes, please refer to the changelog.

This release includes bug fixes and improvements.

## Updated the minimum required SDK and Flutter versions

The minimum required Flutter version has been updated from 3.6.0 to 3.9.0.
The minimum required Dart SDK version has been updated from 3.27.1 to 3.3.5.0.

The version update was necessary to better reflect the minimum versions required by dependencies and to align with the best practices enforced by the `flutter_lints` package.

## Deprecated the *area* and *areaSecond* members from the *MapViewRenderInfo* class

The *area* and *areaSecond* members of the *MapViewRenderInfo* class have been deprecated. These members were previously used to represent the geographical area visible on the map.
Issues are present in scenarios where the map is rotated or tilted, leading to inaccuracies in area representation.

The newly introduced *polygonArea* member provides a more accurate representation of the visible area on the map, especially when the map is rotated as the `PolygonGeographicArea` is better suited to represent areas whose sides are not parallel to the map axes.

Before:
```dart
MapViewRenderInfo info = ...;
RectangleGeographicArea area = info.area;
```

After:
```dart
MapViewRenderInfo info = ...;
PolygonGeographicArea area = info.polygonArea;
```

Issues still exist in scenarios where the map is zoomed too far out, near the poles or the antimeridian. Accurate results can be achieved at the city level and beyond.
For this reason, the newly added `polygonArea` member is marked as experimental.

## Enum value changes in *MapExtendedCapability* enum

The `scenicRoutingAttributes` and `utf8Strings` values were added to the `MapExtendedCapability` enum.
If you use the `MapExtendedCapability` enum in a switch statement, you need to add cases for the new enum values.

## *VehicleRegistration* is now base class for *ElectricBikeProfile* and *MotorVehicleProfile*

The `VehicleRegistration` class is now a base class for `ElectricBikeProfile` and `MotorVehicleProfile` (and indirectly for `CarProfile` and `TruckProfile`).
This change should not require any code changes.

These classes now have an additional `plateNumber` property, which is optional and can be set during instantiation.
