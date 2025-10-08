---
description: Documentation for Get Started Navigation
title: Get Started Navigation
---

# Get started with Navigation

The Maps SDK for Flutter provides developers with comprehensive tools to build a robust turn-by-turn navigation system. This functionality enables applications to track the current device location relative to a predefined route and deliver real-time navigational guidance.

Key Features:

- **Turn-by-Turn Directions**: Provides detailed route instructions based on the device’s current location, ensuring accurate navigation.

- **Live Guidance**: Navigation instructions can be delivered as text and integrated with a Text-to-Speech (TTS) system for voice-based guidance.

- **Warning Alerts**: A versatile alert system that notifies users of conditions such as speed limits, traffic reports, and other important events along the route.

- **Offline Functionality**: Essential navigation features remain operational offline, provided that map data has been pre-downloaded or cached.

The turn-by-turn navigation system relies on continuous acquisition of device data, including location, speed, and heading. These data points are matched against the mapped route and used to generate accurate guidance for the user. Instructions are dynamically updated as the user progresses along the route.

In the event that the user deviates from the planned route, the system will notify them of their off-track distance and provide the option for route recalculations or updates. Additionally, the system can dynamically adjust the route based on real-time traffic conditions, offering more efficient and faster alternatives to optimize navigation.

Additionally, developers can leverage a built-in location simulator to test navigation functionalities during the app development phase.

## How does it work?

To enable navigation, the first step is to compute a valid, navigable route (note that non-navigable routes, such as range routes, are not supported).

The SDK offers two methods for navigating a route:

- **Navigation:** This method relies on the position data provided by the PositionService to guide the user along the route.

- **Simulation:** This method does not require user-provided position data. Instead, it simulates the navigation instructions that would be delivered to the user, allowing developers to test and preview the experience without needing an actual position.

If we are in navigation mode, the position is provided by `PositionService`.It can use:

- **Real GPS Data:** When ``PositionService.setLiveDataSource`` is called, the service will use real-time GPS data to provide position updates. This requires the appropriate application permissions, which differ between Android and iOS. Additionally, the application must programmatically request these permissions from the user.

- **Custom Position Data:** In this mode, a custom data source can be configured to supply position updates. No permissions are required in this case, as the positions are provided through the custom source rather than the device's GPS. If you want to use a custom position take a look at [Custom positioning](/guides/positioning/custom-positioning).

Currently, only one  of navigation and simulation can be active at a time, regardless of the number of maps present within the application.

## Starting a navigation

Given that a route has been computed, the simplest way to navigate it is by implementing the following code:
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

The `NavigationService.startNavigation` method returns `null` only when the geographic search fails to initialize. In such cases, calling `NavigationService.cancelNavigation(taskHandler)` is not possible. Error details will be delivered through the `onError` callback function of the `NavigationService.startNavigation` method.

We declared a function that handles the main navigation events:

- a navigation error (you can see a detailed table below).

- destination is reached .

- a new instruction is available.

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

The navigation can be stopped at any moment or it will be stopped when we reach the destination.

Typically (optional), before starting navigation, we instruct the mapController to begin following the user's position.
More information about the ``startFollowingPosition`` method and related customization options can be found inside the [Show your location on the map](/guides/positioning/show-your-location-on-the-map) guide.

To enhance navigation clarity, the route is displayed on a map. This also includes turn-by-turn navigation arrows that disappear once the user has passed them. More about presenting routes [here](/guides/maps/display-map-items/display-routes).

Navigating on said route will change color of parsed route portion with ``traveledInnerColor`` parameter of ``RouteRenderSettings``.

## Starting a simulation

To start a simulation, you can use the following approach:
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

When simulating we can specify a `speedMultiplier` to set the simulation speed (1.0 is default and corresponds to the maximum speed limit for each road segment). See ``simulationMinSpeedMultiplier`` and ``simulationMaxSpeedMultiplier`` for the range of allowed values.

## Listen for navigation events

A wide range of navigation-related events can be monitored.
In the previous examples, we demonstrated handling the ``onNavigationInstructionUpdated`` event.

Additionally, it is possible to listen for and handle numerous other events:
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

Most callbacks from the table provided above can be used for both simulation and navigation. The `onSkipNextIntermediateDestinationDetected` and `onTurnAround` methods do not make sense on a simulation and will not be called.

When getting the `onSkipNextIntermediateDestinationDetected()` notification it makes sense to drop the first intermediary waypoint. This can be done like this (can make sense also in other situations):
```dart
NavigationService.skipNextIntermediateDestination();
```

## Data source based navigation

Navigation typically relies on the current GPS position. However, it is also entirely valid to perform navigation using custom-defined positions.

This this can be done by creating a custom data source, setting the position service to the given data source, starting the data source and starting navigation as you would with live data source.

See the [custom positioning guide](/guides/positioning/custom-positioning) for more information on how to create a custom data source.

## Stop navigation/simulation

The ``cancelNavigation`` method from the ``NavigationService`` class can be used to stop both navigations and simulations, passing the ``TaskHandler`` returned by the ``startSimulation`` and ``startNavigation`` methods. At the moment it is not possible to pause the simulation.

After stopping the simulation the data source used in the position service is set back to the previous data source if it exists.

## Export Navigation instruction

The `exportAs` method serializes the current **navigation instruction** into a `Uint8List` data buffer.  
For now, the only supported format is `PathFileFormat.packedGeometry`. 
```dart
  final Uint8List bytes = instruction.exportAs(fileFormat: PathFileFormat.packedGeometry);
```

`exportAs` only works with **`PathFileFormat.packedGeometry`**. Passing any other value will return an empty Uint8List. 

## Relevant examples demonstrating navigation related features

- [Navigate route](/examples/routing-navigation/navigate-route)

- [Simulate navigation](/examples/routing-navigation/simulate-navigation)

- [Lane instruction](/examples/routing-navigation/lane-instruction)

- [Speed watcher](/examples/routing-navigation/speed-watcher)

- [External position service navigation](/examples/routing-navigation/external-position-source-navigation)
