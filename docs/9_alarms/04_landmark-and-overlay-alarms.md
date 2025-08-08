---
description: Documentation for Landmark And Overlay Alarms
title: Landmark And Overlay Alarms
---

# Landmark and overlay alarms

The AlarmService can be configured to send notifications upon approaching specific landmarks or overlay items within a defined proximity. This behavior can be tailored to trigger notifications exclusively during navigation or simulation modes, or while freely exploring the map without a predefined route.

This can be used to implement different use cases such as:

- Notify users about incoming reports such as speed cameras, police, accidents or other road hazards.

- Notify users when approaching points of interest, such as historical landmarks, monuments, or scenic viewpoints.

- Notify users about traffic signs such as stop and give way signs.

You can search for landmarks along the active route, whether in navigation or simulation mode, using specific categories or other criteria. Once identified, these landmarks can be added for monitoring. Be sure to account for potential route deviations.

## Configure the alarm distance

The distance threshold measured in meters for triggering notifications when approaching a landmark or overlay item can be obtained as follows:
```dart
double alarmDistance = alarmService.alarmDistance;
```

This threshold can also be adjusted as needed. For example, the following snippet will configure the alarm service to start sending notifications when within 200 m of one of the monitored items:
```dart
alarmService.alarmDistance = 200;
```

## Configure alarms without active navigation

The AlarmService can be configured to control whether notifications about approaching landmarks or overlay items are triggered exclusively during navigation (or navigation simulation) on a predefined route, or if they should also occur while freely exploring the map without an active navigation session.

The following snippet demonstrates how to enable notifications for monitored landmarks at all times, regardless of whether navigation is active:
```dart
alarmService.monitorWithoutRoute = true;
```

The value of the preference set above can be accessed as follows:
```dart
bool isMonitoringWithoutRoute = alarmService.monitorWithoutRoute;
```

## Landmark alarms

### Configure alarms listeners

Users are notified through the `onLandmarkAlarmsUpdated` and `onLandmarkAlarmsPassedOver` callbacks when approaching the specified landmarks and when the landmarks have been passed respectively.

The following snippet demonstrates how to retrieve the next landmark to be intercepted and the distance to it and how to detect when a landmark has been passed:
```dart
AlarmListener alarmListener = AlarmListener(
    onLandmarkAlarmsUpdated: () {
        // The landmark alarm list containing the landmarks that are to be intercepted
        LandmarkAlarmsList landmarkAlarms = alarmService!.landmarkAlarms;

        // The landmarks and their distance from the reference position
        // Sorted ascending by distance from the current position
        List<LandmarkPosition> items = landmarkAlarms.items;

        // The closest landmark and its associated distance (in meters)
        LandmarkPosition closestLandmark = items.first;
        Landmark landmark = closestLandmark.landmark;
        int distance = closestLandmark.distance;

        showSnackbar("The landmark ${landmark.name} is $distance meters away");
    },
    onLandmarkAlarmsPassedOver: () {
        showSnackbar("Landmark was passed over");

        // The landmarks that were passed over
        LandmarkAlarmsList landmarkAlarmsPassedOver = alarmService!.landmarkAlarmsPassedOver;

        // Process the landmarks that were passed over ...
    },
);
```

The ``onLandmarkAlarmsUpdated`` callback will be continuously triggered once the threshold distance is exceeded, until the landmark is intercepted.
The ``onLandmarkAlarmsPassedOver`` callback is called once when the landmark is intercepted.

The items within ``landmarkAlarmsPassedOver`` do not have a predefined sort order. To identify the most recently passed landmark, you can compare the current list of items with the previous list of ``landmarkAlarmsPassedOver`` entries.

### Specify the landmarks to be monitored

Landmarks can be defined and added to the AlarmService for monitoring. The user will receive notifications through the `onLandmarkAlarmsUpdated` callback when approaching specified landmarks.
```dart
// Create landmarks to monitor
Landmark landmark1 = Landmark()
    ..name = "Landmark 1"
    ..coordinates = Coordinates(latitude: 49.0576, longitude: 1.9705);

Landmark landmark2 = Landmark()
    ..name = "Landmark 2"
    ..coordinates = Coordinates(latitude: 43.7704, longitude: 1.2360);

// Create landmark store and add the landmarks
LandmarkStore landmarkStore = LandmarkStoreService.createLandmarkStore("Landmarks to be monitored");
landmarkStore.addLandmark(landmark1);
landmarkStore.addLandmark(landmark2);

// Monitor the landmarks
alarmService.landmarks.add(landmarkStore);
```

Multiple ``LandmarkStore`` instances can be added to the monitoring list simultaneously. These stores can be updated later by modifying the landmarks within them or removed from the monitoring list as needed.

## Overlay alarms

The workflow for overlay items is similar to that for landmarks, with comparable behavior and functionality. Notifications for overlay items are triggered in the same way as for landmarks, based on proximity or other criteria. The steps for monitoring, and handling events related to overlay items align with those for landmarks. All the notices specified above for landmarks are also applicable for overlay items.

To enable overlay alarms, a GemMap widget must be created, and a style containing the overlay to be monitored should be applied. Additionally, the overlay needs to be enabled for the alarms to function properly.

### Configure alarms listeners

The following snippet demonstrates how to retrieve the next overlay item to be intercepted and the distance to it and how to detect when a overlay item has been passed:
```dart
AlarmListener alarmListener = AlarmListener(
    onOverlayItemAlarmsUpdated: () {
        // The overlay item alarm list containing the overlay items that are to be intercepted
        OverlayItemAlarmsList overlayItemAlarms =
            alarmService!.overlayItemAlarms;

        // The overlay items and their distance from the reference position
        // Sorted ascending by distance from the current position
        List<OverlayItemPosition> items = overlayItemAlarms.items;

        // The closest overlay item and its associated distance
        OverlayItemPosition closestOverlayItem = items.first;
        OverlayItem overlayItem = closestOverlayItem.overlayItem;
        int distance = closestOverlayItem.distance;

        showSnackbar("The overlay item ${overlayItem.name} is $distance meters away");
    },
    onOverlayItemAlarmsPassedOver: () {
        // The overlay items that were passed over
        OverlayItemAlarmsList overlayItemAlarmsPassedOver = alarmService!.overlayItemAlarmsPassedOver;

        // Process the overlay items that were passed over ...

        showSnackbar("Overlay item was passed over");
    },
);
```

### Specify the overlays to be monitored

The workflow for specifying the overlay items to be monitored is a bit different from the workflow for landmarks. Instead of specifying the landmarks one by one, the overlay items are specified as a whole, based on the overlay and on the overlay categories.

The snippet below shows how to add the social reports overlay (containing police reports and road hazards) to be monitored:
```dart
int socialReportsOverlayId = CommonOverlayId.socialReports.id;
alarmService.overlays.add(socialReportsOverlayId);
```

## Relevant example demonstrating overlay alarms related features

- [Route Alarms](/examples/routing-navigation/route-alarms)
