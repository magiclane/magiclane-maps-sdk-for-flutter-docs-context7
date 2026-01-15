---
description: Documentation for Get Started Navigation
title: Get Started Navigation
---

# Get started with Navigation

This guide shows you how to implement turn-by-turn navigation in your Flutter app.

**What you need**

- A computed route (non-navigable routes like range routes are not supported)

- Proper location permissions for real GPS navigation

- Map data downloaded for offline functionality

**Key features:**

- **Turn-by-Turn Directions** - Detailed route instructions based on current location

- **Live Guidance** - Text and voice instructions via Text-to-Speech integration

- **Warning Alerts** - Speed limits, traffic reports, and route events

- **Offline Support** - Works offline with pre-downloaded map data

The navigation system tracks your device location, speed, and heading, matching them against the route to generate accurate guidance. Instructions update dynamically as you progress.

When you deviate from the route, the system notifies you and offers recalculation options. It can also adjust routes based on real-time traffic for faster alternatives.

You can test navigation features using the built-in location simulator during development.

---

## How navigation works

The SDK offers two navigation methods:

- **Navigation** - Uses position data from `PositionService` to guide users along the route

- **Simulation** - Simulates navigation instructions without real position data for testing

Navigation mode uses `PositionService` with:

- **Real GPS Data** - Call `PositionService.setLiveDataSource` to use real-time GPS. Requires location permissions on Android and iOS

- **Custom Position Data** - Configure a custom data source for position updates. No permissions required. See [Custom positioning](/guides/positioning/custom-positioning)

Only one navigation or simulation can be active at a time, regardless of map count.

---

## Start navigation

Once you have a computed route, start navigation with this code:
```dart
void navigationInstructionUpdated(NavigationInstruction instruction,
    Set<NavigationInstructionUpdateEvents> events) {
  for (final event in events) {
    switch (event) {
      case NavigationInstructionUpdateEvents.nextTurnUpdated:
        showSnackbar("Turn updated");
        break;
      case NavigationInstructionUpdateEvents.nextTurnImageUpdated:
        showSnackbar("Turn image updated");
        break;
      case NavigationInstructionUpdateEvents.laneInfoUpdated:
        showSnackbar("Lane info updated");
        break;
    }
  }
  final instructionText = instruction.nextTurnInstruction;
  // handle instruction
}

void onDestinationReached(Landmark destination) {
  // handle destination reached
}

void onError(GemError err) {
  // handle error
}
//highlight-start
TaskHandler? handler = NavigationService.startNavigation(route,
    onNavigationInstruction: navigationInstructionUpdated,
    onDestinationReached: onDestinationReached,
    onError: onError);
//highlight-end
// [Optional] Set the camera to follow position.
// Usually we want this when in navigation mode
mapController.startFollowingPosition();
// At any moment, we can cancel the navigation
// NavigationService.cancelNavigation(taskHandler);
```

The `NavigationService.startNavigation` method returns `null` only when the geographic search fails to initialize. In such cases, calling `NavigationService.cancelNavigation(taskHandler)` is not possible. Error details are delivered through the `onError` callback.

The code declares functions that handle main navigation events:

- Navigation errors (see table below)

- Destination reached

- New instructions available

The `err` provided by the callback function can have the following values:
<table>
<tr>
<th>Value</th>
<th>Significance</th>
</tr>
<tr>
<td>`GemError.success`</td>
<td>successfully completed</td>
</tr>
<tr>
<td>`GemError.cancel`</td>
<td>cancelled by the user</td>
</tr>
<tr>
<td>`GemError.waypointAccess`</td>
<td>couldn't be found with the current preferences</td>
</tr>
<tr>
<td>`GemError.connectionRequired`</td>
<td>if allowOnlineCalculation = false in the routing preferences and the calculation can't be done on the client side due to missing data</td>
</tr>
<tr>
<td>`GemError.expired`</td>
<td>calculation can't be done on client side due to missing necessary data and the client world map data version is no longer supported by the online routing service</td>
</tr>
<tr>
<td>`GemError.routeTooLong`</td>
<td>routing was executed on the online service and the operation took too much time to complete (usually more than 1 min, depending on the server overload state)</td>
</tr>
<tr>
<td>`GemError.invalidated`</td>
<td>the offline map data changed ( offline map downloaded, erased, updated ) during the calculation</td>
</tr>
<tr>
<td>`GemError.noMemory`</td>
<td>routing engine couldn't allocate the necessary memory for the calculation</td>
</tr>
</table>

Navigation stops when you reach the destination or cancel it manually.

Before starting navigation, instruct the `mapController` to follow the user's position. See [Show your location on the map](/guides/positioning/show-your-location-on-the-map) for customization options.

Display the route on the map for better navigation clarity. Turn-by-turn navigation arrows disappear once passed. Learn more in [Display routes](/guides/maps/display-map-items/display-routes).

The traveled portion of the route changes color using the `traveledInnerColor` parameter of `RouteRenderSettings`.

---

## Start simulation

Start a simulation with this code:
```dart
void simulationInstructionUpdated(NavigationInstruction instruction,
    Set<NavigationInstructionUpdateEvents> events) {
  for (final event in events) {
    switch (event) {
      case NavigationInstructionUpdateEvents.nextTurnUpdated:
        showSnackbar("Turn updated");
        break;
      case NavigationInstructionUpdateEvents.nextTurnImageUpdated:
        showSnackbar("Turn image updated");
        break;
      case NavigationInstructionUpdateEvents.laneInfoUpdated:
        showSnackbar("Lane info updated");
        break;
    }
  }
  final instructionText = instruction.nextTurnInstruction;
  // handle instruction
}

mapController.preferences.routes.add(route, true);

//highlight-start
TaskHandler? taskHandler = NavigationService.startSimulation(
  route,
  onNavigationInstruction: simulationInstructionUpdated,
  speedMultiplier: 2,
);
//highlight-end
// [Optional] Set the camera to follow position.
// Usually we want this when in navigation mode
mapController.startFollowingPosition();
// At any moment, we can cancel the navigation
// NavigationService.cancelNavigation(taskHandler);
```

The `speedMultiplier` sets simulation speed (default is 1.0, matching the maximum speed limit for each road segment). Check `simulationMinSpeedMultiplier` and `simulationMaxSpeedMultiplier` for allowed values.

---

## Listen for navigation events

You can monitor various navigation events. The previous examples showed handling the `onNavigationInstructionUpdated` event.

Here are additional events you can handle:
```dart
void onNavigationInstruction(NavigationInstruction navigationInstruction,
    Set<NavigationInstructionUpdateEvents> events) {}

void onNavigationStarted() {}

void onTextToSpeechInstruction(String text) {}

void onWaypointReached(Landmark landmark) {}

void onDestinationReached(Landmark landmark) {}

void onRouteUpdated(Route route) {}

void onBetterRouteDetected(
    Route route, int travelTime, int delay, int timeGain) {}

void onBetterRouteRejected(GemError error) {}

void onBetterRouteInvalidated() {}

void onSkipNextIntermediateDestinationDetected() {}

void onTurnAround() {}

void onRouteCalculationStarted() {}

void onRouteCalculationCompleted(GemError error) {}

TaskHandler? taskHandler = NavigationService.startNavigation(
  route,
  onNavigationInstruction: onNavigationInstruction,
  onNavigationStarted: onNavigationStarted,
  onTextToSpeechInstruction: onTextToSpeechInstruction,
  onWaypointReached: onWaypointReached,
  onDestinationReached: onDestinationReached,
  onRouteUpdated: onRouteUpdated,
  onBetterRouteDetected: onBetterRouteDetected,
  onBetterRouteRejected: onBetterRouteRejected,
  onBetterRouteInvalidated: onBetterRouteInvalidated,
  onSkipNextIntermediateDestinationDetected: onSkipNextIntermediateDestinationDetected,
  onTurnAround: onTurnAround,
  onRouteCalculationStarted: onRouteCalculationStarted,
  onRouteCalculationCompleted: onRouteCalculationCompleted,
);
```

These events are described in the following table:

<table>
  <tr>
    <th>Event</th>
    <th>Explanation</th>
  </tr>
  <tr>
    <td>onNavigationInstruction(NavigationInstruction navigationInstruction, Set&lt;NavigationInstructionUpdateEvents&gt; events)</td>
    <td>Triggered when a new navigation instruction is available, providing details about the instruction and offers additional information regarding the reason the event was triggered (valuable for optimizing UI redraws), information accessible in the ``NavigationInstructionUpdateEvents`` enum.</td>
  </tr>
  <tr>
    <td>onNavigationStarted()</td>
    <td>Called when navigation begins, signaling the start of the route guidance.</td>
  </tr>
  <tr>
    <td>onTextToSpeechInstruction(String text)</td>
    <td>Provides a text string for a maneuver that can be passed to an external text-to-speech engine for audio guidance.</td>
  </tr>
  <tr>
    <td>onWaypointReached(Landmark landmark)</td>
    <td>Invoked when a waypoint in the route is reached, including details of the waypoint.</td>
  </tr>
  <tr>
    <td>onDestinationReached(Landmark landmark)</td>
    <td>Called upon reaching the final destination, with information about the destination landmark.</td>
  </tr>
  <tr>
    <td>onRouteUpdated(Route route)</td>
    <td>Fired when the current route is updated, providing the new route details.</td>
  </tr>
  <tr>
    <td>onBetterRouteDetected(Route route, int travelTime, int delay, int timeGain)</td>
    <td>Triggered when a better alternative route is detected, including the new route and details such as travel time, delays caused by traffic and time gains. See te [Better route detection guide](better-route-detection) for more details.</td>
  </tr>
  <tr>
    <td>onBetterRouteRejected(GemError error)</td>
    <td>Called when a check for better routes fails, with details of the rejection error. Used especially for debugging.</td>
  </tr>
  <tr>
    <td>onBetterRouteInvalidated()</td>
    <td>Indicates that a previously suggested better route is no longer valid.</td>
  </tr>
  <tr>
    <td>onSkipNextIntermediateDestinationDetected()</td>
    <td>Indicates we are getting away from the first intermediary waypoint. If this is received, it could be a good moment to call `NavigationService.skipNextIntermediateDestination()`.</td>
  </tr>
  <tr>
    <td>onTurnAround()</td>
    <td>Called when user travel direction violates a link one way restriction or after a navigation route recalculation, if the new route is heading on the opposite user travel direction.</td>
  </tr>
  <tr>
    <td>onRouteCalculationStarted()</td>
    <td>Called when a route recalculation is initiated. Also gets called at the start of the navigation process.</td>
  </tr>
  <tr>
    <td>onRouteCalculationCompleted(GemError error)</td>
    <td>Called when a route recalculation is completed, providing details about any errors that occurred related to route calculation. Also gets called at the start of the navigation process. See the [calculate routes guide](../routing/get-started-routing#calculate-routes) for information about the possible error values. The route is passed via the `onRouteUpdated` callback</td> 
  </tr>
</table>

Most callbacks work for both simulation and navigation. The `onSkipNextIntermediateDestinationDetected` and `onTurnAround` methods are not called during simulation.

When you receive `onSkipNextIntermediateDestinationDetected()`, drop the first waypoint:
```dart
NavigationService.skipNextIntermediateDestination();
```

---

## Use custom data sources

Navigation typically uses GPS position, but you can also use custom-defined positions.

Create a custom data source, set the position service to it, start the data source, and begin navigation as you would with live data.

See the [Custom positioning guide](/guides/positioning/custom-positioning) for details.

---

## Stop navigation or simulation

Use the `cancelNavigation` method from `NavigationService` to stop navigation or simulation. Pass the `TaskHandler` returned by `startSimulation` or `startNavigation`. Pausing simulation is not currently supported.

After stopping simulation, the position service reverts to the previous data source if one exists.

---

## Export navigation instructions

The `exportAs` method serializes the current navigation instruction into a `Uint8List` data buffer. Currently, only `PathFileFormat.packedGeometry` is supported.
```dart
final Uint8List bytes = instruction.exportAs(fileFormat: PathFileFormat.packedGeometry);
```

`exportAs` only works with `PathFileFormat.packedGeometry`. Other values return an empty `Uint8List`.

---

## Run navigation in background

To use navigation while your app is in the background, additional setup is required for iOS and Android.

See the [Background location guide](../positioning/background-location) for configuration instructions.

---

## Relevant examples demonstrating navigation related features

- [Navigate route](/examples/routing-navigation/navigate-route)

- [Simulate navigation](/examples/routing-navigation/simulate-navigation)

- [Lane instruction](/examples/routing-navigation/lane-instruction)

- [Speed watcher](/examples/routing-navigation/speed-watcher)

- [External position service navigation](/examples/routing-navigation/external-position-source-navigation)
