---
description: Documentation for Migrate To 2 22 0
title: Migrate To 2 22 0
---

# Migrate to 2.22.0

This guide outlines the breaking changes introduced in SDK version 2.22.0. Required updates may vary depending on your use case.

Additionally, new features and bugfixes have been introduced and are **not** documented here. For a comprehensive list of changes, please refer to the changelog.

This release adds support for conversion between different types of projections and includes various fixes. Listening for audio events is also available.

## Changes made between different SDK sessions are now cleared

Any changes made before `GemKit.release` is called no longer impact the current session. Once `GemKit.release` is invoked, the session is effectively terminated. When `GemKit.initialize` is called again, a new session begins with a clean slate, meaning it starts with an empty state and none of the previous modifications carry over.

Before:
```dart
// First session
await GemKit.initialize(...);
SdkSettings.mapLanguage = MapLanguage.nativeLanguage;
await GemKit.release();

// Second session
await GemKit.initialize(...);
print(SdkSettings.mapLanguage) // <-- Prints MapLanguage.nativeLanguage, set from the first session
```

After:
```dart
// First session
await GemKit.initalize(...);
SdkSettings.mapLanguage = MapLanguage.nativeLanguage;
await GemKit.release();

// Second session
await GemKit.initalize(...);
print(SdkSettings.mapLanguage) // <-- Prints MapLanguage.automaticLanguage, which is the default value for the field
```

This enables the creation of independent unit tests and integration tests that run without affecting each other.

Ensure that all instances of `GemMap` are removed from the widget tree prior to invoking the `GemKit.release()` method; failure to do so may result in application crashes.

## The return type of the *getTopicNotificationsServiceRestriction* method from the *SdkSettings* class has changed

The method returned a single `OnlineRestrictions` value. It now returns a set of `OnlineRestrictions` values.

Before:
```dart
OnlineRestrictions restriction = SdkSettings.getTopicNotificationsServiceRestriction(ServiceGroupType.mapDataService);
```

After:
```dart
Set<OnlineRestrictions> restriction = SdkSettings.getTopicNotificationsServiceRestriction(ServiceGroupType.mapDataService);
```

Multiple restrictions might be available for a service at any given time. For example, it is possible to have both `OnlineRestrictions.connection` and `OnlineRestrictions.authorization`.

## The *cancelNavigation* method of the *NavigationService* class can now be called with no parameters

The previously positional mandatory `taskHandler` parameter is now optional. When called with no value or null value it cancels the currently active navigation.
This change makes canceling the active navigation easier and simplifies the state management required for this operation.

This change should not involve changes in customer code.

## The *distance* method of the *Coordinates* class also takes an optional *ignoreAltitude* named parameter

The newly introduced named parameter `ignoreAltitude` allows the caller to explicitly ignore altitude differences when calculating the distance between two coordinates.
This parameter defaults to `false` to preserve the existing behavior, which includes altitude in the computation.

This change makes it easier to compute surface-level distances without requiring workaround logic and does not involve changes in customer code.

## The return type of the *getLogMetadata* method in the *RecorderBookmarks* class is now nullable

The method getLogMetadata previously returned a non-nullable `LogMetadata` value. It now returns a nullable `LogMetadata?`.

When passed with invalid `logPath` parameter it returns `null`.
When passed with a valid `logPath` value it return a valid non-empty `LogMetadata` value.

Before this change the method threw exception on invalid input.

Before:
```dart
RecorderBookmarks bookmarks = ...
String path = ...

try {
    LogMetadata logMetadata = bookmarks.getLogMetadata(logPath);
    // Do something with logMetadata
} catch (e) {
    // Handle the case where logMetadata could not be created
}
```

After:
```dart
RecorderBookmarks bookmarks = ...
String path = ...

LogMetadata? logMetadata = bookmarks.getLogMetadata(logPath);
if (logMetadata != null) {
    // Do something with logMetadata
} else {
    // Handle the case where logMetadata could not be created
}
```

## The *scenic* value has been added to the *RouteType* enum

When the `scenic` option is selected in the route computation preferences, the engine calculates the fastest route that also offers the most scenic views between the specified waypoints.

This change should not involve changes in customer code.
