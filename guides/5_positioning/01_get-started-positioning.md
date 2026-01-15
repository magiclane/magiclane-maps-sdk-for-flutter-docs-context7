---
description: Documentation for Get Started Positioning
title: Get Started Positioning
---

# Get started with positioning

The Positioning module provides location data for navigation, tracking, and location-based services. Use GPS data from the device or integrate custom location data from external sources.

---

## Grant permissions

### Step 1: Add application location permissions

Configure location permissions based on the platform:

<Tabs>
<TabItem value="android" label="Android" default>
    Add the following permissions to `android/app/main/AndroidManifest.xml` within the `<manifest>` block:
    
```xml
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />
    ```

    More information about permissions can be found in the [Android Manifest documentation](https://developer.android.com/reference/android/Manifest.permission).

</TabItem>
<TabItem value="ios" label="IOS" default>
    Add the following lines to `ios/Runner/Info.plist` within the `<dict>` block:

    
```xml
    <key>NSLocationWhenInUseUsageDescription</key>
    <string>Location is needed for map localization and navigation</string>
    ```

    Add the following to the `ios/Podfile`:

    
```ruby
    post_install do |installer|
        installer.pods_project.targets.each do |target|
            flutter_additional_ios_build_settings(target)

            target.build_configurations.each do |config|
            config.build_settings['GCC_PREPROCESSOR_DEFINITIONS'] ||= [
                '$(inherited)',
                ## dart: [PermissionGroup.location, PermissionGroup.locationAlways, PermissionGroup.locationWhenInUse]
                'PERMISSION_LOCATION=1',
            ]
            end 
        end
    end
    ```

</TabItem>
</Tabs>

### Step 2: Get user consent

Use the [permission_handler](https://pub.dev/packages/permission_handler) plugin to request and manage location permissions:

Follow the [platform specific setup](https://pub.dev/packages/permission_handler#setup) for the permission_handler package to correctly configure permissions for your application.
```dart
// For Android & iOS platforms, permission_handler package is used to ask for permissions.
final locationPermissionStatus = await Permission.locationWhenInUse.request();

if (locationPermissionStatus == PermissionStatus.granted) {
    // After the permission was granted, we can set the live data source (in most cases the GPS).
    // The data source should be set only once, otherwise we'll get GemError.exist error.
    GemError setLiveDataSourceError = PositionService.setLiveDataSource();
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

The code requests the `locationWhenInUse` permission. If granted, call `setLiveDataSource` on the `PositionService` instance to use real GPS positions for navigation.

When the live data source is set and permissions are granted, the position cursor appears on the map as an arrow.

For debugging, use the Android Emulator's [extended controls](https://developer.android.com/studio/run/emulator-extended-controls) to mock the current position. On real devices, use apps like [Mock Locations](https://play.google.com/store/apps/details?id=ru.gavrikov.mocklocations) to simulate location data.

On iOS (simulators and devices), mock the location from Xcode for GPX replay. See more [here](https://developer.apple.com/documentation/xcode/simulating-location-in-tests).

If the location permission popup doesn't appear, verify the platform-specific setup is correct. Check the `permission_handler` package documentation and issue tracker.

---

## Receive location updates

Register a callback function using the `addImprovedPositionListener` and `addPositionListener` methods to receive position updates. The listener is called continuously as new position data becomes available.

Consult the [Positions guide](/guides/core/positions) for more information about the `GemPosition` class and the differences between raw positions and map-matched positions.

### Raw positions

Use the `addPositionListener` method to listen for raw position updates as they're pushed to the data source or received via sensors:
```dart
PositionService.addPositionListener((GemPosition position) {
    // Process the position
});
```

### Map-matched positions

Use the `addImprovedPositionListener` method to register a callback for map-matched positions:
```dart
PositionService.addImprovedPositionListener((GemImprovedPosition position) {
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

During simulation, the positions provided through the `addImprovedPositionListener` and `addPositionListener` methods correspond to the simulated locations generated as part of the navigation simulation process.

---

## Get current location

Use the `position` getter from the `PositionService` class to retrieve the current location. This returns a `GemPosition` object with the latest location information or `null` if no position data is available. Use this to access the most recent position without registering for continuous updates:
```dart
GemPosition? position = PositionService.position;

if (position == null) {
    showSnackbar("No position");
} else {
    showSnackbar("Position: ${position.coordinates}");
}
```

A similar getter is provided for map-matched positions: `improvedPosition` returns `GemImprovedPosition?` instead of `GemPosition?`.

---

## Relevant examples demonstrating positioning related features

- [Follow Position](/examples/maps-3dscene/follow-position)

- [Custom Position Icon](/examples/maps-3dscene/custom-position-icon)

- [External Position Source Navigation](/examples/routing-navigation/external-position-source-navigation)
