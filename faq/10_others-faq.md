---
description: Documentation for Others Faq
title: Others Faq
---

# Others FAQ

<sub style={{ color: 'gray', display: 'inline-block' }}>
  
</sub>

# Is it possible to obtain speed limit information based on the current position while there is no navigation started?

Yes, it is possible to obtain speed limit listening to improved position using PositionService. The `GemImprovedPosition` class has a `speed` and a `speedLimit` getter. See the [positioning guide](/guides/positioning/get-started-positioning) for more details.

Also check the [speed warning](/guides/alarms/speed-alarms) guide for more details about speed warning.

## How can I enable/disable the traffic layer visibility on map?

By default, the traffic layer is visible (unless it is disabled in the map style). You can disable the traffic layer programmatically:
```dart
_controller.preferences.setTrafficVisibility(false);
```

## Why I do not have a speed limit?

Some road sections might not have speed limit data or, in some cases, there is no speed limit (e.g. Autobahn).
In this situation, the `speedLimit` will equal 0.

## Why satellite view doesn't show up when I initially start the application?

Satellite data is accessible only when the device is connected to the internet and cannot be cached for offline use, unlike map data. If the satellite data is not fully loaded, a preview version will be displayed temporarily.
