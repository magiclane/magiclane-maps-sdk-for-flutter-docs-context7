---
description: Documentation for Get Started Positioning
title: Get Started Positioning
---

# Get started with positioning

The Positioning module enables your application to obtain and utilize location data, serving as the foundation for features like navigation, tracking, and location-based services. This data can be sourced either from the device's GPS or from a custom data source, offering flexibility to suit diverse application needs.

Using the Positioning module, you can:

- Leverage real GPS data: Obtain highly accurate, real-time location updates directly from the device's built-in GPS sensor.

- Integrate custom location data: Configure the module to use location data provided by external sources, such as mock services or specialized hardware.

In the following sections, you will learn how to grant the necessary location permissions for your app, set up live data sources, and manage location updates effectively. Additionally, we will explore how to customize the position tracker to align with your application's design and functionality requirements. This comprehensive guide will help you integrate robust and flexible positioning capabilities into your app.

## Granting permissions

### Add application location permissions

Location permissions must be configured based on the platform the application is running on. Below are the steps to enable location permissions for Android and iOS:

<Tabs>
<TabItem value="android" label="Android" default>
    In order to give the application the location permission on Android, you need to alter the `android/app/main/AndroidManifest.xml` and add the following permissions within the `<manifest>` block:
    
```xml
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />
    ```

    More information about permissions can be found in the  [Android Manifest documentation](https://developer.android.com/reference/android/Manifest.permission).

</TabItem>
<TabItem value="ios" label="IOS" default>
    On iOS, you need to adapt the `ios/Runner/Info.plist` by adding the following lines of code within the `<dict>` block:

    
```xml
    <key>NSLocationWhenInUseUsageDescription</key>
    <string>Location is needed for map localization and navigation</string>
    ```

</TabItem>
</Tabs>

### Get user consent

Defining location permissions in the application, as explained in the previous section, is a mandatory step.

Afterward, we can utilize the [permission_handler](https://pub.dev/packages/permission_handler) plugin to programmatically request and manage location permissions within the application:

Ensure that you follow the [platform specific setup](https://pub.dev/packages/permission_handler#setup) for the permission_handler package to correctly configure permissions for your application.
```dart
// For Android & iOS platforms, permission_handler package is used to ask for permissions.
final locationPermissionStatus = await Permission.locationWhenInUse.request();

if (locationPermissionStatus == PermissionStatus.granted) {
    // After the permission was granted, we can set the live data source (in most cases the GPS).
    // The data source should be set only once, otherwise we'll get GemError.exist error.
    GemError setLiveDataSourceError = PositionService.instance.setLiveDataSource();
    showSnackbar("Set live datasource with result: $setLiveDataSourceError");
}

if (locationPermissionStatus == PermissionStatus.denied) {
    // The user denied the permission
    showSnackbar("Location permission denied");
}

if (locationPermissionStatus == PermissionStatus.permanentlyDenied) {
    // The user permanently denied the permission
    // The user should go to the app settings to enable the permission
    showSnackbar("Location permission permanently denied");
}
```

In the previous code, we request the `locationWhenInUse` permission. If the user grants it, we notify the engine to use real GPS positions for navigation by calling `setLiveDataSource` on the `PositionService` instance.

If the live data source is set and the right permissions are granted, the position cursor should be visible on the map as an arrow.

:::tip[tip]

For debugging purposes, the Android Emulator's [extended controls](https://developer.android.com/studio/run/emulator-extended-controls) can be used to mock the current position.
On a real device, apps such as [Mock Locations](https://play.google.com/store/apps/details?id=ru.gavrikov.mocklocations) can be utilized to simulate location data.

On iOS (simulators and devices) the location can be mocked from XCode, allowing for GPX replay. See more [here](https://developer.apple.com/documentation/xcode/simulating-location-in-tests). 

## Receive location updates

To receive updates about changes in the current position, we can register a callback function using the ``addImprovedPositionListener`` and ``addPositionListener`` methods. The given listener will be called continuously as new updates about current position are made.

Consult the [Positions guide](/guides/core/positions) for more information about the `GemPosition` class and the differences between raw positions and map matched position.

### Raw positions

To listen for raw position updates (as they are pushed to the data source or received via the sensors), the ``addPositionListener`` method can be used:
```dart
PositionService.instance.addPositionListener((GemPosition position) {
    // Process the position
});
```

### Map matched positions

The ``addImprovedPositionListener`` method is used to register a callback to receive map matched positions.
```dart
PositionService.instance.addImprovedPositionListener((GemImprovedPosition position) {
    // Current coordinates
    Coordinates coordinates = position.coordinates;
    print("New position: ${coordinates}");

    // Speed in m/s (-1 if not available)
    double speed = position.speed;

    // Speed limit in m/s on the current road (0 if not available)
    double speedLimit = position.speedLimit;

    // Heading angle in degrees (N=0, E=90, S=180, W=270, -1 if not available)
    double course = position.course;

    // Information about current road (if it is in a tunnel, bridge, ramp, one way, etc.)
    Set<RoadModifier> roadModifiers = position.roadModifiers;

    // Quality of the current position
    PositionQuality fixQuality = position.fixQuality;

    // Horizontal and vertical accuracy in meters
    double accuracyHorizontal = position.accuracyH;
    double accuracyVertical = position.accuracyV;
});
```

During simulation, the positions provided through the ``addImprovedPositionListener`` and ``addPositionListener`` methods correspond to the simulated locations generated as part of the navigation simulation process.

## Get current location

To retrieve the current location, we can use the ``position`` getter from the ``PositionService`` class. This method returns a ``GemPosition`` object containing the latest location information or ``null`` if no position data is available. This method is useful for accessing the most recent position data without registering for continuous updates.
```dart
GemPosition? position = PositionService.instance.position;

if (position == null) {
    showSnackbar("No position");
} else {
    showSnackbar("Position: ${position.coordinates}");
}
```

A similar getter is provided for map-matched positions: `improvedPosition` and returns `GemImprovedPosition?` instead of `GemPosition?`.

## Relevant examples demonstrating positioning related features

- [Follow Position](/examples/maps-3dscene-examples/follow-position)

- [Custom Position Icon](/examples/maps-3dscene-examples/custom-position-icon)

- [External Position Source Navigation](/examples/routing-navigation/external-position-source-navigation)
