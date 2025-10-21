---
description: Documentation for Better Route Detection
title: Better Route Detection
---

# Better route detection

The Maps SDK for Flutter continuously monitors traffic conditions and automatically evaluates alternative routes to ensure optimal navigation. This feature enhances user experience by providing real-time route adjustments, reducing travel time, and improving overall efficiency, especially in dynamic traffic environments.

## Requirements

#### Route preferences

For this feature to function, the route used in navigation or simulation must be computed with specific settings within the `RoutePreferences` object:

- the `transportMode` needs to be `RouteTransportMode.car` or `RouteTransportMode.lorry`

- the `avoidTraffic` needs to be `TrafficAvoidance.all` or `TrafficAvoidance.roadblocks`

- the `routeType` needs to be `RouteType.fastest`

Unless the settings are set as above the better route detection feature will not trigger.
```dart
final routePreferences = RoutePreferences(
    routeType: RouteType.fastest,
    avoidTraffic: TrafficAvoidance.roadblocks,
    transportMode: RouteTransportMode.car,
);
```

Additional settings can be configured within the `RoutePreferences` object during route calculation, as long as they do not override or conflict with the required preferences mentioned above.

#### Traffic

For the callbacks to be triggered, traffic needs to be present of the route on which the navigation is active. 

#### Significant time gain

A newly identified route must have a substantial time delay compared to the current route for it to be considered. The relevant callback will only be triggered if an alternative route offers a time savings of more than five minutes, ensuring that route adjustments are meaningful and beneficial to the user.

The better route detection feature will not function as intended if any of the required conditions outlined above are not met.

## Listen for notification events

The `startSimulation` and `startNavigation` methods provided by the `NavigationService` class enable the registration of the following callbacks:

- `onBetterRouteDetected` : Triggered when a better route is identified. It provides information such as the newly detected route, its total travel time, the traffic-induced delay on the new route, and the time savings compared to the current route.

- `onBetterRouteInvalidated` : Triggered when a previously detected better route is no longer valid. This can occur if the user deviates from the shared trunk of both routes, an even better alternative becomes available, or changing traffic conditions eliminate the previously detected advantage.

- `onBetterRouteRejected` : Triggered when no suitable alternative route is found during the better route check.

It is the responsibility of the API user to manage the recommended route. The navigation service does not automatically switch to the better route, requiring explicit handling and implementation by the user.
```dart
NavigationService.startSimulation(
    route,
    onBetterRouteDetected:(route, travelTime, delay, timeGain) {
        print("A better route has been detected - total travel time: $travelTime s, traffic delay on the better route: $delay s, time gain from current route: $timeGain s");
        // Do something with the route ...
    },
    onBetterRouteInvalidated: (){
        print("The previously found better route is no longer valid");
    },
    onBetterRouteRejected: (reason) {
        print("The check for better route failed with reason: $reason");
    },
);
```

## Force the check for better route

The system automatically performs the better route check at predefined intervals, provided all required conditions are met.

Additionally, the API user can manually trigger the check by calling the `checkBetterRoute` static method from the `Debug` class. If a better route is found, the `onBetterRouteDetected` callback is invoked; otherwise, if no suitable alternative is available, the `onBetterRouteRejected` callback is triggered—assuming the check was successfully initiated.
```dart
Debug.checkBetterRoute();
```

## Relevant examples demonstrating better route related features

- [Better Route Notification](/examples/routing-navigation/better-route-notification)
