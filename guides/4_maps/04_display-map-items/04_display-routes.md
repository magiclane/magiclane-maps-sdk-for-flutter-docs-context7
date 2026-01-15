---
description: Documentation for Display Routes
title: Display Routes
---

# Display routes

Learn how to display routes on the map, customize their appearance, and manage route labels.

---

## Add routes to the map

Display routes on the map using `MapViewPreferences.routes.add(route, isMainRoute)`. Multiple routes can be displayed simultaneously, but only one can be the **main route**, with others treated as secondary. Specify the main route by passing `true` to the `bMainRoute` parameter when calling `MapViewRoutesCollection.add`, or use the `MapViewRoutesCollection.mainRoute` setter.
```dart
mapController.preferences.routes.add(route, true);
mapController.centerOnRoute(route);
```

To center on a route with padding, refer to the [Adjust Map View](/guides/maps/adjust-map#center-on-an-area-with-padding) guide. Utilize the `screenRect` parameter in the `centerOnRoute` method to define the specific region of the viewport that should be centered.
```dart
mapController.preferences.routes.add(route1, true);
mapController.preferences.routes.add(route2, false);
mapController.preferences.routes.add(route3, false);
mapController.centerOnMapRoutes();
```

---

## Customize route appearance

Customize route appearance using `RouteRenderSettings` when adding the route via the `routeRenderSettings` optional parameter, or later using the `MapViewRoute.renderSettings` setter.
```dart
final renderSettings = RouteRenderSettings(innerColor: Color.fromARGB(255, 255, 0, 0));
mapController.preferences.routes.add(route, true, routeRenderSettings: renderSettings)
```


```dart
final mapViewRoute = mapController.preferences.routes.getMapViewRoute(0);

mapViewRoute?.renderSettings = RouteRenderSettings(innerColor: Color.fromARGB(255, 255, 0, 0));
```

All dimensional sizes within the `RouteRenderSettings` are measured in millimeters.

---

## Remove routes

Remove all displayed routes using `MapViewRoutesCollection.clear()`. To remove only secondary routes while keeping the main route, use `mapController.preferences.routes.clearAllButMainRoute()`.

---

## Add route labels

Routes can include labels that display information such as ETA, distance, toll prices, and more. Attach a label to a route using the `label` optional parameter of the `MapViewRoutesCollection.add` method:

### Add icons to labels

Enhance labels by adding up to **two icons** using the optional `labelIcons` parameter, which accepts a `List`. Access available icons through the `GemIcon` enum.
```dart
    controller.preferences.routes.add(routes.first, true,
        label: "This is a custom label",
        labelIcons: [
    SdkSettings.getImgById(GemIcon.favoriteHeart.id)!,
    SdkSettings.getImgById(GemIcon.waypointFinish.id)!,
]);
```

### Auto-generate labels

Auto-generate labels using the `autoGenerateLabel` parameter:
```dart
mapController.preferences.routes.add(route, true, autoGenerateLabel: true);
```

Enabling `autoGenerateLabel` will override any customizations made with the `label` and `labelIcons` parameters.

### Hide labels

Hide a route label by calling `MapViewRoutesCollection.hideLabel(route)`. You can also manage labels through a `MapViewRoute` object using the `labelText` setter to assign a label or the `hideLabel` method to hide it.

---

## Check visible route portion

Retrieve the visible portion of a route—defined by its start and end distances in meters—using the `getVisibleRouteInterval` method from the `GemMapController`:
```dart
final (startRouteVisibleIntervalMeters, endRouteVisibleIntervalMeters) = controller.getVisibleRouteInterval(route);
```

You can provide a custom screen region to the `getVisibleRouteInterval` method instead of using the entire viewport. The method returns `(0,0)` if the route is not visible on the provided viewport or region.

---

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
