---
description: Documentation for Migrate To 2 18 0
title: Migrate To 2 18 0
---

# Migrate to 2.18.0

This guide outlines the breaking changes introduced in SDK version 2.18.0. Required updates may vary depending on your use case.

Additionally, new features and bugfixes have been introduced and are **not** documented here. For a comprehensive list of changes, please refer to the changelog.

This release introduces enhancements to the `OffBoardListener` class, improving event handling for new content versions and the auto-update mechanism.

## The `setAllowConnection` method from the `SdkSettings` class has been deprecated

The `setAllowConnection` method from the `SdkSettings` has been deprecated. In its place the following features should be used:

    - The `setAllowInternetConnection` from the `SdkSettings` class in order to allow/deny internet connection for the whole SDK.

    - The `register...` methods from the `SdkSettings.offBoardListener` member in order to specify callbacks. This approach allows the user to specify callbacks individually, whithout overriding exising callbacks.

    - The `isAutoUpdateForResourcesEnabled` setter of the `SdkSettings.offBoardListener` member replaces the `canDoAutoUpdateResources` parameter.

Before:
```dart
SdkSettings.setAllowConnection(
    true,
    canDoAutoUpdateResources : false,
    onWorldwideRoadMapSupportStatusCallback: 
        onWorldwideRoadMapSupportStatusCallback: (status) {
        // Do something with the status...
    },
);
```

After:
```dart
SdkSettings.offBoardListener.registerOnWorldwideRoadMapSupportStatus((status){
    // Do something with the status... 
}),

// Might not be needed, depending on the usecase and the previously set configuration within the offboardListener
SdkSettings.offBoardListener.isAutoUpdateForResourcesEnabled = false;
SdkSettings.setAllowInternetConnection(true);
```

The auto-update mechanism is no longer overwriten when registering callbacks via the `setAllowConnection`/`offBoardListener.register.../GemKit.initialize/GemMap()` methods / objects.

The newly added `SdkSettings.offBoardListener` member allows changing the current auto update settings at any time and listening for when the auto-update has been completed. See the [Update Content page](../offline/update-content) for more details.
