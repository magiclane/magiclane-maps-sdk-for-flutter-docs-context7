---
description: Documentation for Migrate To 3 1 6
title: Migrate To 3 1 6
---

# Migrate to 3.1.6

This guide outlines the breaking changes introduced in SDK version 3.1.6. Required updates may vary depending on your use case.

Additionally, new features and bugfixes have been introduced and are **not** documented here. For a comprehensive list of changes, please refer to the changelog.

This release focuses on API surface cleanup, getter consistency, and clearer platform permission ownership to keep projects stable when upgrading to 3.1.5.

## The *RoutingAlgoModifiers* enum and related *Debug* members have been removed

Affected members include:

- `RoutingAlgoModifiers`

- `Debug.getRoutingAlgoModifiers`

- `Debug.setRoutingAlgoModifiers`

- `Debug.routingAlgoModifiers`

These members were designed to be internal-only and are no longer part of the public API. Remove any usage of them; there is no public replacement.

## The *isFollowingPositionTouchHandlerModified* and *isDefaultFollowingPosition* methods are now getters

Affected members include:

- `GemMapController.isFollowingPositionTouchHandlerModified`

- `GemMapController.isDefaultFollowingPosition`

- `GemView.isFollowingPositionTouchHandlerModified`

- `GemView.isDefaultFollowingPosition`

Update method calls to property access by removing the parentheses.

Before:
```dart
final isModified = controller.isFollowingPositionTouchHandlerModified();
final isDefault = view.isDefaultFollowingPosition();
```

After:
```dart
final isModified = controller.isFollowingPositionTouchHandlerModified;
final isDefault = view.isDefaultFollowingPosition;
```

## The return type of *NavigationInstruction.getRoadInfoImg* is now *RoadInfoImg*

Update any variables or method signatures that expect `Img` as result of a `getRoadInfoImg` call to use `RoadInfoImg` instead.

Before:
```dart
Img img = instruction.getRoadInfoImg(...);
```

After:
```dart
RoadInfoImg img = instruction.getRoadInfoImg(...);
```

The `RoadInfoImg` class provides more flexibility for retrieving road info images with resizable dimensions and custom background colors.

## The return type of *SocialReportsOverlayInfo.getSocialReportsCategory* is now *SocialReportsOverlayCategory?*

Update your types and null checks to use `SocialReportsOverlayCategory?` instead of `OverlayCategory?`.

Before:
```dart
OverlayCategory? category = info.getSocialReportsCategory();
```

After:
```dart
SocialReportsOverlayCategory? category = info.getSocialReportsCategory();
```

The `SocialReportsOverlayCategory` class is a subtype of `OverlayCategory` with additional social reports specific functionality.

## Android Bluetooth and phone state permissions are no longer added by default

Affected members include:

- `BLUETOOTH`

- `BLUETOOTH_CONNECT`

- `READ_PHONE_STATE`

- `MODIFY_AUDIO_SETTINGS`

If your app requires these permissions, add them explicitly in your Android manifest. This change reduces default permission scope and keeps your manifest aligned with your app’s actual needs.

Add the following lines to your `AndroidManifest.xml` if needed:
```xml
<uses-permission android:name="android.permission.BLUETOOTH" />
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
<uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS" />
```

This change help better align permissions with Google Play app requirements and improves user trust by minimizing unnecessary permission requests.
