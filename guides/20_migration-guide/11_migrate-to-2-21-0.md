---
description: Documentation for Migrate To 2 21 0
title: Migrate To 2 21 0
---

# Migrate to 2.21.0

This guide outlines the breaking changes introduced in SDK version 2.21.0. Required updates may vary depending on your use case.

Additionally, new features and bugfixes have been introduced and are **not** documented here. For a comprehensive list of changes, please refer to the changelog.

This release overhauls the `SocialOverlay` class, adds support for storing preferences as key-value pairs, and includes various fixes.

## Overhauls in the *SocialOverlay* class

### Changes regarding the *report* method

All parameters of the `report` method are now named.
The `description`, `snapshot`, `format`, and `parameters` arguments are now optional. 

The return type has changed from `GemError` to `EventHandler?`. If the operation can be started, a non-null `EventHandler` is returned. If the operation cannot be started, `null` is returned.  

Errors are now reported through the newly added `onComplete` callback.  
The `GemError.scheduled` error is no longer used; instead, `GemError.success` is returned when the operation completes.

Before:
```dart
GemError res = SocialOverlay.report(
    idReport,
    subCategory.uid,
    "TEST MAGIC LANE",
    image,
    ImageFileFormat.png,
    params,
);
print("Report result error: $res"); // <-- It is the error associated with the start of the operation. Does not indicate if the operation is successful.
```

After:
```dart
EventHandler? handler = SocialOverlay.report(
    prepareId: idReport,
    categId: subCategory.uid,
    description: "TEST MAGIC LANE",
    snapshot: image,
    format: ImageFileFormat.png,
    onComplete: (GemError error) {
        print("Report result error: $error"); // <-- It is the error associated with the start of the operation. GemError.success indicates the operation is successful.
    },
);
```

### Changes regarding other asynchronous methods

The `updateReport`, `confirmReport`, `denyReport`, `deleteReport`, and `addComment` methods have been changed:  

- The return type is now `EventHandler?` instead of `GemError`. The value `null` is returned when the operation could not be started.  

- The `onComplete` callback parameter has been added, now providing the result of the operation. The `GemError.scheduled` error is no longer used. Instead, the `GemError.success` value is triggered after the operation has completed.

Before:
```dart
GemError error = SocialOverlay.updateReport(item: overlays.first, params: params);
```

After:
```dart
EventHandler? handler = SocialOverlay.updateReport(
    item: overlays.first,
    params: params,
    onComplete: (GemError error) {
        print("Update result error: $error");
    },
);
```

## Some getters are now nullable in public transit related classes

The type of the `publicTransportFare` getter of the `PTRoute` class is `String?` instead of `String`.
The type of the `stopPlatformCode` getter of the `PTTrip` class is `String?` instead of `String`.

The value null is returned when the related information is not available for an object.

Before:
```dart
PTRoute route = ...

String publicTransportFare = route.publicTransportFare;
if (publicTransportFare.isEmpty){
    // Fare info not available
}
```

After:
```dart
PTRoute route = ...

String? publicTransportFare = route.publicTransportFare;
if (publicTransportFare == null){
    // Fare info not available
}
```

## The *autoPlaySound* parameter of the *startSimulation* and *startnavigation* method of the *NavigationService* class is now nullable

The `autoPlaySound` parameter is now nullable. When set to `null` (the new default), the SDK preserves the previous configuration regarding TTS instruction playback. Previously, `autoPlaySound` defaulted to `false`, which disabled TTS playback.

The `canPlaySounds` getter/setter pair in the `SoundPlayingService` class can now be used to control TTS instruction playback independently of starting a new navigation session. As a result, the `autoPlaySound` is now deprecated.

Before:
```dart
NavigationService.startNavigation(
    ....
    autoPlaySound : true,
);
```

After:
```dart
NavigationService.startNavigation(
    ....
);

SoundPlayingService.canPlaySounds = true;
```

This allows the `canPlaySounds` setting to be toggled before, during, or after a navigation session.

## The *dispose* method of the *LandmarkStore* class is now sync and no longer needs to be awaited

Before:
```dart
LandmarkStore store = ...
await store.dispose();
```

After:
```dart
LandmarkStore store = ...
store.dispose();
```

## The *automaticTimestamp* field of the *RoutePreferences* class has been removed

Setting the `timestamp` field to `null` now has the same effect as setting `automaticTimestamp` to `true`.

The `automaticTimestamp` field has been removed to avoid discrepancies between it and `timestamp`, which could lead to confusing results in public transit scheduling.

Before:
```dart
final preferences = RoutePreferences(
    automaticTimestamp: true,
    ...
);
```

After:
```dart
final preferences = RoutePreferences(
    timestamp: null, // <-- Can also be omitted as the default timestamp value is null
    ...
);
```

