---
description: Documentation for Background Location
title: Background Location
---

# Background Location

Some use cases require location access when the app is in the background. To enable this, configure both **iOS** and **Android** platforms with the appropriate permissions and services.

Enable background location support for features like recording, navigation, or content download (maps can be large and take time to fetch).

---

## Configure iOS

Update your `Info.plist` to request permission for background location access:
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

---

## Configure Android

Declare permissions in your manifest and implement a foreground service to keep location updates alive.

### Add required permissions

Include the necessary permissions and service declarations in your `AndroidManifest.xml`:
```xml
<uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />
<!-- Needed for foreground service -->
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_LOCATION" />
<uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
```

On Android 10+ (API level 29+), request **ACCESS_BACKGROUND_LOCATION** explicitly at runtime in your app's permission request flow.

### Implement a foreground service

Android requires a foreground service to run background location tracking. Without it, the operating system may terminate your app's background processes, leading to loss of location updates.

Create a foreground service on Android to ensure location updates continue when the app is in the background.

Learn more about foreground services: [Background Location Example](https://developer.android.com/guide/components/foreground-services).

Find a sample implementation in the [Recorder in Background example](/examples/routing-navigation/background-recorder-with-foreground-service).

---

## Configure sensor settings

Enable background location in the SDK by initializing the sensor configuration in your Flutter code:
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

---

## Relevant examples demonstrating background location related features

- [Recorder In Background](/examples/routing-navigation/background-recorder-with-foreground-service)
