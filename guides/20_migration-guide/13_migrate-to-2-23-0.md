---
description: Documentation for Migrate To 2 23 0
title: Migrate To 2 23 0
---

# Migrate to 2.23.0

This guide outlines the breaking changes introduced in SDK version 2.23.0. Required updates may vary depending on your use case.

Additionally, new features and bugfixes have been introduced and are **not** documented here. For a comprehensive list of changes, please refer to the changelog.

This release improves the handling of invalid input and invalid results and brings uniformity to the API. As such, there are many breaking changes introduced.

## Many methods and getters now return nullable types when results are unavailable

To improve reliability and make nullability explicit in your code, several methods and getters across the SDK have been updated to return nullable types (`T?` instead of `T`) when their result may be absent due to invalid input or unavailable data.

This change helps prevent unintended behavior caused by assuming values are always available. By signaling the potential for `null`, developers are now encouraged to perform proper null checks, which improves code safety, readability, and debugging clarity.

We apologize for the breaking nature of this change. While it may require small adjustments in your codebase (e.g., adding null checks or fallback values), it ultimately leads to safer and more predictable application behavior.

Affected methods and classes:

| Class                   | Method / Getter          | Old return type         | New return type          | Observation                                                                                                                                                                         |
|-------------------------|--------------------------|-------------------------|--------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Debug                   | getRouteConnections      | MarkerCollection        | MarkerCollection?        | Returns null if the connections could not be built                                                                                                                                  |
| DriverBehaviourAnalysis | drivingScores            | DrivingScores           | DrivingScores?           | Returns null if the scores are not available for the current session                                                                                                                 |
| DriverBehaviour         | stopAnalysis             | DriverBehaviourAnalysis | DriverBehaviourAnalysis? | Returns null if the analysis could not be computed (the sessions was too short/insufficient data/ etc)                                                                                |
| DriverBehaviour         | getInstantaneousScores   | DrivingScores           | DrivingScores?           | Returns null if the scores are not available for the current session (insufficient data)                                                                                              |
| DriverBehaviour         | getInstantaneousScores   | DrivingScores           | DrivingScores?           | Returns null if the scores are not available for the current session                                                                                                                 |
| DriverBehaviour         | getCombinedAnalysis      | DriverBehaviourAnalysis | DriverBehaviourAnalysis? | Returns null if the combined analysis is not available (no previous analysis)                                                                                                        |
| NavigationInstruction   | nextNextTurnDetails      | TurnDetails             | TurnDetails?             | Returns null if there is no next next turn or if the turn does not provide extra data                                                                                               |
| NavigationInstruction   | nextTurnDetails          | TurnDetails             | TurnDetails?             | Returns null if there is no next turn or if the turn does not provide extra data                                                                                                    |
| NavigationInstruction   | signpostDetails          | SignpostDetails         | SignpostDetails?         | Returns null if there is are no signpost details available                                                                                                                          |
| NavigationService       | getNavigationInstruction | NavigationInstruction   | NavigationInstruction?   | Returns null if there is no current instruction (for example if there is no navigation session ongoing)                                                                             |
| LogMetadata             | getUserMetadata          | Uint8List               | Uint8List?               | Returns null or empty buffer if there is no metadata with the given key                                                                                                             |
| RouteInstruction        | toPTRouteInstruction     | PTRouteInstruction      | PTRouteInstruction?      | Returns null if the RouteInstruction is not convertible to a public transit route instruction (the route is not computed using PT settings or if the parent segment is not common ) |
| RouteSegment            | toPTRouteSegment         | PTRouteSegment          | PTRouteSegment?          | Returns null if the RouteSegment is not convertible to a public transit route segment (the route is not computed using PT settings or if the segment is not common )                |
| PTRoute                 | getBuyTicketInformation  | PTBuyTicketInformation  | PTBuyTicketInformation?  | Returns null if the index is invalid                                                                                                                                                |
| OTRoute                 | track                    | Path                    | Path?                    | Returns null if no path is available                                                                                                                                                |
| PTRouteSegment          | getAlert                 | PTAlert                 | PTAlert?                 | Returns null if the index is invalid                                                                                                                                                |
| PTAlert                 | getUrlTranslation        | PTTranslation           | PTTranslation?           | Returns null if the index is invalid                                                                                                                                                |
| PTAlert                 | getUrlTranslation        | PTTranslation           | PTTranslation?           | Returns null if the index is invalid                                                                                                                                                |
| PTAlert                 | getHeaderTextTranslation | PTTranslation           | PTTranslation?           | Returns null if the index is invalid                                                                                                                                                |
| RouteBookmarks          | getPreferences           | RoutePreferences        | RoutePreferences?        | Returns null if the index is invalid                                                                                                                                                |

Before introducing explicit nullable return types, the SDK relied on a mix of implicit mechanisms to signal invalid or missing data. These approaches often led to confusion, required additional logic, or were error-prone. Below is a summary of the previous behaviors for affected classes and methods:

- `getRouteConnections` returned an empty `MarkerCollection` in case of failure.

- `DrivingScores` used `-1` values for individual scores to signal missing or invalid data.  

- There was no simple way to check if a `DriverBehaviourAnalysis` is valid.  

- `hasNextTurnInfo` and `hasNextNextTurnInfo` were used to check for the presence of `TurnDetails` (these are still available).  

- `hasSignpostInfo` was used to check for the presence of `SignpostDetails` (still supported).  

- There was no simple way to check for the existence of a valid `NavigationInstruction`.  

- `getUserMetadata` returned an empty `Uint8List` when metadata was missing.

- There was no easy way to validate whether a `RouteInstruction` or `RouteSegment` was convertible to a PT equivalent (`PTRouteInstruction` / `PTRouteSegment`).  

- Methods returning `PTBuyTicketInformation`, `PTAlert`, and `PTTranslation` required manual index checks to avoid accessing invalid data.  

- `track` returned an empty `Path` in case of failure.

- `getPreferences` returned a default-initialized `RoutePreferences` object for invalid indices, with no way to detect the error. The index needed to be checked manually before using the method.  

To accommodate the new nullable return types, update your code by adding appropriate null checks and handling the absence of data according to your use case.

Before:
```dart
RouteBookmarks routeBookmarks = ...
RoutePreferences prefs = routeBookmarks.getPreferences(5);

// Do something with the preferences...
```

After:
```dart
RouteBookmarks routeBookmarks = ...
RoutePreferences? prefs = routeBookmarks.getPreferences(5);

if (prefs != null) {
    // Treat the case when the return is null
} else {
    // Do something with the preferences...
}
```

## Pair has been replaced with records

All uses of the SDK's generic `Pair` classes have been replaced with Dart's built-in records.

Records offer a concise, type-safe, and idiomatic alternative for representing fixed-size groups of values. This change reduces boilerplate and improves code clarity by leveraging native language features.

Affected methods and classes:

| Class                     | Method (or getter/setter)                              | Changed                             |
|---------------------------|--------------------------------------------------------|-------------------------------------|
| ContentStore              | getStoreContentList                                    | Return type                         |
| ContentStore              | createContentUpdater                                   | Return type                         |
| MapView                   | getVisibleRouteInterval                                | Return type                         |
| MapCamera                 | generatePositionAndOrientation                         | Return type                         |
| MapCamera                 | generatePositionAndOrientationHPR                      | Return type                         |
| MapCamera                 | generatePositionAndOrientationTargetCentered           | Return type                         |
| MapCamera                 | generatePositionAndOrientationRelativeToCenteredTarget | Return type                         |
| MapCamera                 | generatePositionAndOrientationRelativeToTarget         | Return type                         |
| MapDetails                | getSunriseAndSunset                                    | Return type                         |
| FollowPositionPreferences | touchHandlerModifyHorizontalAngleLimits                | Return type & setter parameter type |
| FollowPositionPreferences | touchHandlerModifyVerticalAngleLimits                  | Return type & setter parameter type |
| FollowPositionPreferences | touchHandlerModifyDistanceLimits                       | Return type & setter parameter type |
| FollowPositionPreferences | mapRotationMode                                        | Return type                         |
| OverlayService            | getAvailableOverlays                                   | Return type                         |
| RouteTerrainProfile       | getElevationSamples                                    | Return type                         |
| RouteTerrainProfile       | getElevationSamplesByCount                             | Return type                         |
| RouteTrafficEvent         | fromLandmark                                           | Return type                         |
| RouteTrafficEvent         | toLandmark                                             | Return type                         |

Before:
```dart
final FollowPositionPreferences followPositionPreferences = ...;

// Setter
followPositionPreferences.touchHandlerModifyDistanceLimits = Pair<double, double>(50.0, 100.0);

// Getter
final Pair<double, double> result = followPositionPreferences.touchHandlerModifyDistanceLimits;
final double minVal = result.first;
final double maxVal = result.second;
```

After:
```dart
final FollowPositionPreferences followPositionPreferences = ...;

// Setter
followPositionPreferences.touchHandlerModifyDistanceLimits = (50.0, 100.0);

// Getter
final (double, double) result = followPositionPreferences.touchHandlerModifyDistanceLimits;
final double minVal = result.$1;
final double maxVal = result.$2;

// Or simpler 
final (minVal, maxVal) = followPositionPreferences.touchHandlerModifyDistanceLimits;
```

This change simplifies the codebase by removing the need for boilerplate custom `Pair` classes and makes better use of modern Dart language features.

## The *setTouchHandlerModifyVerticalAngleLimits* method has been replaced with *touchHandlerModifyVerticalAngleLimits* setter

The method `setTouchHandlerModifyVerticalAngleLimits` of the `FollowPositionPreferences` class has been replaced with the touchHandlerModifyVerticalAngleLimits setter.

Before:
```dart
FollowPositionPreferences prefs = ...
prefs.setTouchHandlerModifyVerticalAngleLimits(Pair<double, double> (10.5, 20.5));
```

After:
```dart
FollowPositionPreferences prefs = ...
prefs.touchHandlerModifyVerticalAngleLimits = (10.5, 20.5);
```

This change simplifies the API by removing the need for an explicit setter method. Using a getter promotes a more declarative and fluent interface, allowing direct access to the configurable handler object rather than enforcing a separate method call. It also improves readability and aligns with modern best practices for configuration-style APIs.

## Most *GemList* classes are now only for internal use and are no longer exposed on the public API

The following classes are no longer available in the public API:

- `GemList`,

- `GenericIterator`

- `LandmarkList`

- `LandmarkPositionList`

- `OverlayItemList`

- `RouteList`

- `RouteInstructionList`

- `RouteSegmentList`

- `OverlayItemPositionList`

- `MarkerMatchList`

- `MarkerList`

- `TrafficEventList`

- `RouteTrafficEventList`

- `LandmarkCategoryList`

- `ContentStoreItemList`

- `SignpostItemList`

Use dart predefined list instead of SDK's custom `GemList` classes.

## The *getOnlineServiceRestriction* method of the *SdkSettings* returns set of *OnlineRestrictions* values instead of *OnlineRestrictions*

As multiple restrictions can be available at the same time, the `getOnlineServiceRestriction` now returns a `Set<OnlineRestrictions>` containing all applicable restrictions.
The `none` value of the `OnlineRestrictions` enum has been removed. If no service restrictions are applicable then the returned set is empty.

Before:
```dart
OnlineRestrictions restrictions = SdkSettings.getOnlineServiceRestriction(...);
if (restrictions == OnlineRestrictions.none){
    // No restrictions affecting the service
} else {
    // There is a restriction affecting the service - restrictions
}
```

After:
```dart
OnlineRestrictions restrictions = SdkSettings.getOnlineServiceRestriction(...);
if (restrictions.isEmpty){
    // No restrictions affecting the service
} else {
    // There is list of restrictions affecting the service - restrictions
}
```

