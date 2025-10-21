---
description: Documentation for Display Routes
title: Display Routes
---

# Display routes

Routes can be displayed on the map by using ``MapViewPreferences.routes.add(route, isMainRoute)``. Multiple routes can be displayed at the same time, but only one is the main one, the others being treated as secondary. Specifying which one is the main route can be done when calling ``MapViewRoutesCollection.add`` by passing true to the ``bMainRoute`` parameter, or by calling the ``MapViewRoutesCollection.mainRoute`` setter.
```dart
mapController.preferences.routes.add(route, true);
mapController.centerOnRoute(route);
```

To center on a route with padding, refer to the [Adjust Map View](/guides/maps/adjust-map#map-centering-on-area-with-padding) guide. Utilize the `screenRect` parameter in the `centerOnRoute` method to define the specific region of the viewport that should be centered.
```dart
mapController.preferences.routes.add(route1, true);
mapController.preferences.routes.add(route2, false);
mapController.preferences.routes.add(route3, false);
mapController.centerOnMapRoutes();
```

Route appearance on map can be customized via ``RouteRenderSettings`` when added, passed to the ``MapViewRoutesCollection.add``'s optional parameter ``routeRenderSettings``, or later on, via ``MapViewRoute.renderSettings`` setter.
```dart
final renderSettings = RouteRenderSettings(innerColor: Color.fromARGB(255, 255, 0, 0));
mapController.preferences.routes.add(route, true, routeRenderSettings: renderSettings)
```


```dart
final mapViewRoute = mapController.preferences.routes.getMapViewRoute(0);

mapViewRoute?.renderSettings = RouteRenderSettings(innerColor: Color.fromARGB(255, 255, 0, 0));
```

All dimensional sizes within the `RouteRenderSettings` are measured in millimeters.

To remove displayed routes, use ``MapViewRoutesCollection.clear()``. You can also remove all secondary routes with ``mapController.preferences.routes.clearAllButMainRoute()``.

### Set route labels

A route can include a label that provides information such as ETA, distance, toll prices, and more. To attach a label to a route, the label optional parameter ``label`` of the ``MapViewRoutesCollection.add`` method is utilized:
```dart
mapController.preferences.routes.add(route, true, label: "Added label");
```

You can enhance the label by adding up to **two icons** using the optional `labelIcons` parameter, which accepts a `List`. Available icons can be accessed through the `GemIcon` enum.
```dart
    controller.preferences.routes.add(routes.first, true,
        label: "This is a custom label",
        labelIcons: [
    SdkSettings.getImgById(GemIcon.favoriteHeart.id)!,
    SdkSettings.getImgById(GemIcon.waypointFinish.id)!,
]);
```

The label can also be auto-generated like so:
```dart
mapController.preferences.routes.add(route, true, autoGenerateLabel: true);
```

The label of a route added to the collection can be hidden by calling ``MapViewRoutesCollection.hideLabel(route)``.
Labels can also be managed through a ``MapViewRoute`` object. The ``labelText`` setter is used to assign a label, while the ``hideLabel`` method can be used to hide it.

Enabling `autoGenerateLabel` will override any customizations made with the `label` and `labelIcons` parameters.

### Check what portion of a route is visible on a screen region

To retrieve the visible portion of a route—defined by its start and end distances in meters—use the `getVisibleRouteInterval` method from the `GemMapController`:
```dart
final (startRouteVisibleIntervalMeters, endRouteVisibleIntervalMeters) = controller.getVisibleRouteInterval(route);
```

You can also provide a custom screen region to the `getVisibleRouteInterval` method, instead of using the entire viewport.
The method will return `(0,0)` if the route is not visible on the provided viewport/region of the viewport

## Relevant examples demonstrating routing related features

- [Calculate Route](/examples/routing-navigation/calculate-route)

- [Route Profile](/examples/routing-navigation/route-profile)

- [Route Instructions](/examples/routing-navigation/route-instructions)

- [Finger Route](/examples/routing-navigation/finger-route)

- [Range Finder](/examples/routing-navigation/range-finder)

- [GPX Route](/examples/routing-navigation/gpx-route)

- [Truck Profile](/examples/routing-navigation/truck-profile)

- [Public Transit](/examples/routing-navigation/public-transit)

- [Offline Routing](/examples/routing-navigation/offline-routing)

- [Multi Map Routing](/examples/routing-navigation/multimap-routing)
