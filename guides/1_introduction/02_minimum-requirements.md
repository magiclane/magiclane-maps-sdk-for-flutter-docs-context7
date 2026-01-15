---
description: Documentation for Minimum Requirements
title: Minimum Requirements
---

# Minimum requirements

The Maps SDK for Flutter enables the development of applications for Android and iOS. Web support is planned for an upcoming release. Refer to the minimum requirements for each platform outlined below.

## Development machine

Flutter must be correctly installed and configured according to the requirements for each target platform. See the [Flutter Setup Guide](https://docs.flutter.dev/get-started/install) for more information.

The minimum supported Dart version is 3.9.0 and the minimum supported Flutter version is 3.35.1.

It is always advisable to use the latest stable versions of Dart and Flutter to ensure compatibility, performance, and access to the latest features. Additionally, it is best practice to use the Dart version bundled with Flutter to maintain consistency and avoid potential conflicts.

Please note that Maps SDK for Flutter relies on following packages and versions:

- [plugin_platform_interface](https://pub.dev/packages/plugin_platform_interface) (version ^2.1.8)

- [logging](https://pub.dev/packages/logging) (version ^1.0.0) 

- [ffi](https://pub.dev/packages/ffi) (version ^2.0.1).

- [meta](https://pub.dev/packages/meta) (version ^1.15.0).

- [flutter_lints](https://pub.dev/packages/flutter_lints) (version ^6.0.0).

Be aware that compatibility issues may arise if other dependencies in your project require different versions of these libraries.

## Target devices 

Compatibility is assured with both real devices and emulators used for development. For the positioning service, a device with GPS sensor capability is required.

### Android

- minimum Android API level of 27 is required (Android 8.1 Oreo), providing compatibility with over 96.4% of all Android devices currently in use.

### iOS

- minimum version of iOS 14.0 or iPadOS 14.0 is required for compatibility, ensuring broad compatibility across Apple platforms.

A simple compiled app may require approximately 250MB of storage.

Please be aware that on lower-end devices, performance may be reduced due to hardware limitations. This can affect the smoothness of the user interface and overall responsiveness of the application, particularly in areas with complex or resource-intensive scenes.
