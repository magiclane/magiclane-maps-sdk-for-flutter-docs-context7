---
description: Documentation for Migrate To 2 14 0
title: Migrate To 2 14 0
---

# Migrate to 2.14.0

This guide outlines the breaking changes introduced in SDK version 2.14.0. Required updates may vary depending on your use case.

Additionally, new features and bugfixes have been introduced and are **not** documented here. For a comprehensive list of changes, please refer to the changelog.

This release brings fixes to many issues and expands the features provided by already existing methods.

## Yelp related features were removed

The `getYelpPhoneNumber`, `getYelpRating`, `getYelpImagesCount`, `cancelYelpInfo`, `getYelpName`, `getYelpUrl`, `hasYelpInfo`, `getYelpImagePath`, `getYelpAddress` methods from the `ExternalInfo` class were removed as YELP is no longer supported by the SDK.
The `onYelpDataAvailable` callback parameter from the `getExternalInfo` method has also been removed.

## Removed deprecated and no longer supported methods from the *GemMapController* and *GemView* classes

The `registerLongPressCallback`, `registerOnMapAngleUpdateCallback` and `registerOnMapViewMoveStateChangedCallback` methods were renamed to `registerLongPressCallback`, `registerMapAngleUpdateCallback` and `registerMapViewMoveStateChangedCallback`.
See the [Migrate to 3.13.0](./migrate-to-2-13-0) for more details.

The methods related to registering pointer callbacks were removed as they are no longer working. Please use the other methods provided in the `GemMapController` class instead.

## Some methods were renamed

The `getOverlayById` method from the `OverlayCollection` and `OverlayMutableCollection` classes was renamed to `getOverlayByUId` to better emphasize the field name used.

The `getContourGeograficArea` method from the `Landmark` class was renamed to `getContourGeographicArea` as the typo was fixed.

## Changes within the *RouteTrafficEvent* class

### Getters rename

Affected members include `getDelay`, `getLength`, `getImpactZone`, `getReferencePoint`, `getBoundingBox`, `getEventSeverity`, `getPreviewUrl`, `getAffectedTransportMode`, `getStartTime`, `getEndTime`. All these getters have been renamed and the `get` has been removed.

Before:
```dart
RouteTrafficEvent event = ...
TrafficEventImpactZone zone = event.getImpactZone;
```

After:
```dart
RouteTrafficEvent event = ...
TrafficEventImpactZone zone = event.impactZone;
```

### Other changes

The return type of the `toLandmark` getter has been changed to `Pair<Landmark, bool>` instead of `bool`.

The `asyncUpdateToFromData` method now takes a `void Function(GemError err)` callback instead of `ProgressListener`. 

## The *customizeDefPositionTracker* method from the *MapSceneObject* class now returns *GemError* instead of *int*

The value returned is the error in the form of a `GemError` value instead of the error code.

Before:
```dart
MapSceneObject object = ...;
int error = object.customizeDefPositionTracker(...);
if (error == 0){
    print("The position tracker has been customized successfully");
} else {
    print("An error occurred while customizing the position tracker - code $error");
}
```

After:
```dart
MapSceneObject object = ...;
GemError error = object.customizeDefPositionTracker(...);
if (error == GemError.success){
    print("The position tracker has been customized successfully");
} else {
    print("An error occurred while customizing the position tracker - code ${error.code}");
}
```

## The *getField* method from the *AddressInfo* class now returns *String?* instead of *String*

If the field value is not available then the method returns `null` instead of empty `String`.

Before:
```dart
AddressInfo addressInfo = ...;
String country = addressInfo.getField(AddressField.country);
if (country.isEmpty) print("No country found");
```

After:
```dart
AddressInfo addressInfo = ...;
String? country = addressInfo.getField(AddressField.country);
if (country == null) print("No country found");
```

## The *getRenderSettings*, *getMapViewRoute* and *mainRoute* members from the *MapViewRoutesCollection* return a nullable result

The `getRenderSettings` returns `RouteRenderSettings?` instead of `RouteRenderSettings`. The value null is returned when the input to the function is invalid.
The `getMapViewRoute` returns `MapViewRoute?` instead of `MapViewRoute`. The value null is returned when the input to the function is invalid.
The `mainRoute` returns `Route?` instead of `Route`. The value null is returned when the no routes are contained within the collection.

Always check the returned value for `null` before using it.

## Changed optional parameter type of some methods from the *GemMapController* and *GemView* classes to non-nullable.

Affected methods and parameters are:

- the `zoomLevel` parameter of the `startFollowingPosition` method. Changed type from `int?` to `int`. The default value is now `-1`.

- the `duration` parameter of the `setZoomLevel` and `setSlippyZoomLevel` method. Changed type from `int?` to `int`. The default value is now `0`.

- the `displayMode` parameter of the `centerOnRoutes` method. Changed type from `RouteDisplayMode?` to `RouteDisplayMode`. The default value is now `RouteDisplayMode.full`.

This change is unlikely to impact most user code. If `null` was given explicitly as a parameter then it can be omitted.

Before:
```dart
GemMapController controller = ...;
controller.startFollowingPosition(zoomLevel: null);
```

After:
```
GemMapController controller = ...;
controller.startFollowingPosition();
```

## Changes to logging methods

The `isPrintSdkDebugInfoEnabled` property from the `Debug` class has been removed. Instead, new properties have been introduced to provide more granular control over logging:

- `logCreateObject` : Logs object creation.

- `logCallObjectMethod` : Logs method calls on objects.

- `logListenerMethod` : Logs listener method invocations.

- `logLevel` : Allows configuring logging at different levels.

Before:
```dart
Debug.isPrintSdkDebugInfoEnabled = true;
```

After:
```dart
Debug.logCreateObject = true;
Debug.logCallObjectMethod = true;
Debug.logListenerMethod = true;

Debug.logLevel = GemLoggingLevel.all;
```


