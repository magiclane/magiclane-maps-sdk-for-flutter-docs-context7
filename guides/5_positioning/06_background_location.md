---
description: Documentation for Background Location
title: Background Location
---

# Background Location

Some use cases require location access even when the app is in the background. In such cases, you'll need to configure both **iOS** and **Android** platforms appropriately. The SDK supports this scenario, but platform permissions and services must be correctly set up for it to work.

We recommend enabling background location support if your application includes features like recording, navigation, or content download (especially for maps, which can be quite large and may take a while to fetch).

## iOS Setup

On iOS, you’ll need to update your `Info.plist` to request permission for background location access.
```xml
<key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
<string>Location is needed for map localization and navigation.</string>

<key>UIBackgroundModes</key>
<array>
    <!-- Include other modes as needed -->
    <string>location</string>
    <string>processing</string>
</array>

```

## Android Setup

You'll need to declare permissions in your manifest, and then implement a foreground service to keep location updates alive.

### Required AndroidManifest Changes

Make sure you include the necessary permissions and service declarations in your `AndroidManifest.xml`.
```xml
    <uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE_LOCATION" />
    <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
```

On Android 10+ (API level 29+), you also need to request **ACCESS_BACKGROUND_LOCATION** explicitly at runtime. Make sure to handle this in your app's permission request flow.

### Foreground Service Implementation

Android won’t let you run background location tracking unless you create a foreground service. Here you can find an example implementation: [Background Location Example](https://developer.android.com/guide/components/foreground-services)

### Sensor Configuration

To enable background location within our SDK, you’ll also need to initialize the sensor configuration accordingly in your Flutter code.
```dart
    final dataSource = DataSource.createLiveDataSource();

    if (dataSource == null) {
        throw "Error creating data source";
    }

    final config = dataSource.getConfiguration(DataType.position);
    
    config.allowsBackgroundLocationUpdates = true;

    final err = dataSource.setConfiguration(type: DataType.position, config: config);

    if (err != GemError.success) {
        throw "Error setting data source configuration: ${err.toString()}";
    }
```

## Relevant example demonstrating background location related features

- [Recorder In Background](/examples/routing-navigation/background-location)
