---
description: Documentation for Areas Alarms
title: Areas Alarms
---

# Areas alarms

Another powerful use case is triggering operations the moment a user enters or exits a defined **geographic area**.

The Magic Lane Flutter SDK includes a built-in ``AlarmService`` class, making it effortless to configure and manage all your geofence events.

## Add areas to be monitored

Define your geographic areas—`RectangleGeographicArea`, `CircleGeographicArea`, and `PolygonGeographicArea`—then invoke the `monitorArea` method on your `AlarmService` instance:
```dart
final RectangleGeographicArea rect = RectangleGeographicArea(
    topLeft: Coordinates(latitude: 1, longitude: 0.5),
    bottomRight: Coordinates(latitude: 0.5, longitude: 1),
);

final CircleGeographicArea circle = CircleGeographicArea(
    centerCoordinates: Coordinates(latitude: 1, longitude: 0.5),
    radius: 100,
);

final PolygonGeographicArea polygon = PolygonGeographicArea(coordinates: [
    Coordinates(latitude: 1, longitude: 0.5),
    Coordinates(latitude: 0.5, longitude: 1),
    Coordinates(latitude: 1, longitude: 1),
    Coordinates(latitude: 1, longitude: 0.5),
]);

alarmService!.monitorArea(rect, id: 'areaRect');
alarmService!.monitorArea(circle, id: 'areaCircle');
alarmService!.monitorArea(polygon, id: 'areaPolygon');
```

Assign a unique identifier string to each area you monitor—this lets you easily determine which specific zone a user has entered or exited.

## Get a list of monitored areas

Access your active geofences via the `monitoredAreas` getter, which returns a list of `AlarmMonitoredArea` objects, each one reflecting the parameters you provided to `monitorArea`.
```dart
List<AlarmMonitoredArea> monitorAreas = alarmService.monitoredAreas;

for (final monitorArea in monitorAreas){
    final GeographicArea area = monitorArea.area;
    final String id = monitorArea.id;
}
```

When defining a `PolygonGeographicArea`, always “close” the shape by making the first and last coordinates identical. Otherwise, the SDK may return polygons that don’t match the one you provided.

## Unmonitor an area

To remove a monitored area, call the `unmonitorArea` method and pass in the same `GeographicArea` instance you originally supplied to `monitorArea`. This will unregister that zone and stop all related geofence events.
```dart
final RectangleGeographicArea rect = RectangleGeographicArea(
    topLeft: Coordinates(latitude: 1, longitude: 0.5),
    bottomRight: Coordinates(latitude: 0.5, longitude: 1),
);
alarmService!.monitorArea(rect);

alarmService!.unmonitorArea(rect);
```

The `unmonitorAreasByIds` method can be also used by passing the list of ids to be unmonitored:
```dart
alarmService.unmonitorAreasByIds(['firstIdToUnmonitor', 'secondIdToUnmonitor'])
```

## Get notified when the user enters an area:

Attach your `AlarmListener`—including the `onBoundaryCrossed` callback—to your `AlarmService`. This callback returns two arrays: one of area IDs the user has entered, and another of those they’ve exited.
```dart
AlarmListener(
    onBoundaryCrossed: (List<String> entered, List<String> exited) {
        print("ENTERED AREAS: ${entered}");
        print("EXITED AREAS: ${exited}");
    }
);

alarmService = AlarmService(alarmListener);
```

## Get the list of areas where the user is located

Retrieve the zones the user is currently inside by calling the `insideAreas` getter on your `AlarmService` instance:
```dart
List<AlarmMonitoredArea> insideAreas = alarmService.insideAreas;
```

For the insideAreas getter to return a non-empty list, the user must be inside at least one monitored area and must move or change position within that area.

To retrieve the zones the user has exited, call the `outsideAreas` getter on your `AlarmService` instance.

## Relevant example demonstrating areas alarms related features

- [Areas Alarms](/examples/routing-navigation/areas-alarms)
