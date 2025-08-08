---
description: Documentation for Display Route Instructions
title: Display Route Instructions
---

# Display route instructions

Instructions are represented as arrows on the map and can be displayed by using ``GemMapController.centerOnRouteInstruction(instruction, zoomLevel: zoomLevel)``. To obtain a route's instructions, see the [Get the route segments and instructions](../../routing/get-started-routing#get-the-route-segments-and-instructions) section. The following example iterates through all instructions of the first segment of a route and displays each one by centering:
```dart
for (final instruction in route.segments.first.instructions) {
  mapController.centerOnRouteInstruction(instruction, zoomLevel: 75);

  await Future.delayed(Duration(seconds: 3));
}
```

Only one instruction can be displayed at a time. To remove it, the displayed route which contains the instruction must be removed using either ``mapController.preferences.routes.clear`` or by specifying the route with ``mapController.preferences.routes.remove(route)``.
