---
description: Documentation for Better Route Detection
title: Better Route Detection
---

# Better route detection

Monitor traffic conditions and automatically evaluate alternative routes for optimal navigation. This feature provides real-time route adjustments, reducing travel time and improving efficiency in dynamic traffic environments.

---

## What you need

**Prerequisites:**

- Route computed with specific `RoutePreferences` settings

- Active traffic data on the route

- Significant time gain (over 5 minutes) for alternative routes

---

## Step 1: Configure route preferences

Configure the `RoutePreferences` object with the following required settings:

- `transportMode` — `RouteTransportMode.car` or `RouteTransportMode.lorry`

- `avoidTraffic` — `TrafficAvoidance.all` or `TrafficAvoidance.roadblocks`

- `routeType` — `RouteType.fastest`
```dart
final routePreferences = RoutePreferences(
  routeType: RouteType.fastest,
  avoidTraffic: TrafficAvoidance.roadblocks,
  transportMode: RouteTransportMode.car,
);
```

Add additional settings to `RoutePreferences` during route calculation, provided they don't conflict with the required preferences above.

### Additional requirements

**Traffic data:**  
Traffic must be present on the active navigation route for callbacks to trigger.

**Time savings threshold:**  
Alternative routes must offer time savings exceeding five minutes to be considered. This ensures meaningful route adjustments.

❌ Better route detection will not function if the required conditions above are not met.

---

## Step 2: Register notification callbacks

Register callbacks using `startSimulation` or `startNavigation` methods from the `NavigationService` class:

- **`onBetterRouteDetected`** — triggered when a better route is identified. Provides the new route, total travel time, traffic-induced delay, and time savings compared to the current route.

- **`onBetterRouteInvalidated`** — triggered when a previously detected better route is no longer valid. Occurs if the user deviates from the shared trunk, a better alternative appears, or traffic conditions change.

- **`onBetterRouteRejected`** — triggered when no suitable alternative route is found during the check.

You must manually manage and switch to the recommended route. The navigation service does not automatically switch routes.
```dart
NavigationService.startSimulation(
  route,
  onBetterRouteDetected: (route, travelTime, delay, timeGain) {
    print("Better route detected - travel time: $travelTime s, delay: $delay s, time gain: $timeGain s");
    // Handle the route
  },
  onBetterRouteInvalidated: () {
    print("Previously found better route is no longer valid");
  },
  onBetterRouteRejected: (reason) {
    print("Better route check failed: $reason");
  },
);
```

---

## Step 3: Trigger manual checks (optional)

The system automatically checks for better routes at predefined intervals when all conditions are met.

Manually trigger a check using the `checkBetterRoute` static method from the `Debug` class. The `onBetterRouteDetected` callback is invoked if a better route is found; otherwise, `onBetterRouteRejected` is triggered if no suitable alternative exists.
```dart
Debug.checkBetterRoute();
```

---

## Relevant examples demonstrating better route features

- [Better Route Notification](/examples/routing-navigation/better-route-notification)
