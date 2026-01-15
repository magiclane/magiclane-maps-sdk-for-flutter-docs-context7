---
description: Documentation for Positions
title: Positions
---

# Positions

This page covers position data representation using `GemPosition` and `GemImprovedPosition` classes for GPS-based systems.

---

Don't confuse `Coordinates` with `Position` classes. The `Coordinates` class represents geographic locations (latitude, longitude, altitude) and is widely used throughout the SDK. In contrast, `GemPosition` and `GemImprovedPosition` classes contain additional sensor data and primarily represent the user's location and movement details.

## Create positions

Instantiate the `GemPosition` class using methods in the `SenseDataFactory` class. You can also access it through methods exposed by the Maps SDK for Flutter.

For more details, refer to the [Get Started with Positioning](/guides/positioning/get-started-positioning) guide.

---

## Raw position data

Raw position data represents unprocessed GPS sensor data from devices. It corresponds to the `GemPosition` interface.

## Map matched position data

Map matching aligns raw GPS data with a digital map, correcting inaccuracies by snapping the position to the nearest logical location (such as roads). It corresponds to the `GemImprovedPosition` interface.

---

## Compare position types

Map matched positions provide more information than raw positions:

| Attribute               | Raw | Map Matched | When is available     | Description
|------------------------ | --- | ----------- | --------------------- | -------------
| acquisitionTime         | ✅  | ✅          | always                | The system time when the data was collected from sensors.
| satelliteTime           | ✅  | ✅          | always                | The satellite timestamp when position was collected by the sensors.
| provider                | ✅  | ✅          | always                | The provider type (GPS, network, unknown)
| latitude & longitude    | ✅  | ✅          | hasCoordinates        | The latitude and longitude at the position in degrees
| altitude                | ✅  | ✅          | hasAltitude           | The altitude at the given position. Might be negative
| speed                   | ✅  | ✅          | hasSpeed              | The current speed (always non-negative)
| speedAccuracy           | ✅  | ✅          | hasSpeedAccuracy      | The current speed accuracy (always non-negative). Typical accuracy is 2 m/s in good conditions
| course                  | ✅  | ✅          | hasCourse             | The current direction of movement in degrees (0 north, 90 east, 180 south, 270 west)
| courseAccuracy          | ✅  | ✅          | hasCourseAccuracy     | The current heading accuracy is degrees (typical accuracy is 25 degrees)
| accuracyH               | ✅  | ✅          | hasHorizontalAccuracy | The horizontal accuracy in meters. Always positive. (typical accuracy 5-20 meters)
| accuracyV               | ✅  | ✅          | hasVerticalAccuracy   | The vertical accuracy in meters. Always positive.
| fixQuality              | ✅  | ✅          | always                | The accuracy quality (inertial – based on extrapolation, low – inaccurate, high – good accuracy, invalid – unknown)
| coordinates             | ✅  | ✅          | hasCoordinates        | The coordinates of the position
| roadModifiers           | ❌  | ✅          | hasRoadLocalization   | The road modifiers (such as tunnel, bridge, ramp, etc.)
| speedLimit              | ❌  | ✅          | always                | The speed limit on the current road in m/s. It is 0 if no speedLimit information is available
| terrainAltitude         | ❌  | ✅          | hasTerrainData        | The terrain altitude in meters. Might be negative. It can be different than altitude
| terrainSlope            | ❌  | ✅          | hasTerrainData        | The current terrain slope in degrees. Positive values for ascent, negative values for descent.
| address                 | ❌  | ✅          | always                | The current address.

The `speedLimit` field may not always have a value, even if the position is map matched. This can happen if data is unavailable for the current road segment or if the position is not on a road. In such cases, the `speedLimit` field will be set to 0.

To check if a user is exceeding the legal speed limit, use the `AlarmService` class. Refer to the [speed warnings guide](../alarms/speed-alarms) for more details.
