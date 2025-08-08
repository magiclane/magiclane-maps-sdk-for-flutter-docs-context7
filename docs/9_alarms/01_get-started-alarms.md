---
description: Documentation for Get Started Alarms
title: Get Started Alarms
---

# Get started with Alarms

The alarm system within the GemSDK offers a range of monitoring and notification functionalities. It allows for the detection and management of different types of alarms, such as boundary crossings, speed limit violations, and landmark alerts. The system provides the ability to configure parameters like alarm distance, overspeed thresholds, and whether monitoring should occur even without a route being followed.

The key feature of this system is its ability to monitor specific geographic areas or routes and trigger alarms when predefined conditions, such as crossing a boundary or entering/exiting a tunnel, are met. It also provides customization options for alarm behavior based on location (e.g., inside or outside city limits). Additionally, users can set up callbacks to receive notifications about specific events, including changes in monitoring state or when a landmark alarm is triggered.

The system supports interaction with various alarm types, including overlay item and landmark alarms, and offers an easy interface for both setting and getting alarm-related information.

Multiple alarm services and listeners can operate simultaneously, allowing for the monitoring of various events concurrently.

## Configure AlarmService and AlarmListener

The code snippet provided below defines an ``AlarmListener`` and creates an ``AlarmService`` based on this listener:
```dart
// Create alarm listener and specify callbacks
AlarmListener alarmListener = AlarmListener(
    onBoundaryCrossed: (entered, exited) {},
    onMonitoringStateChanged: (isMonitoringActive) {},
    onTunnelEntered: () {},
    onTunnelLeft: () {},
    onLandmarkAlarmsUpdated: () {},
    onOverlayItemAlarmsUpdated: () {},
    onLandmarkAlarmsPassedOver: () {},
    onOverlayItemAlarmsPassedOver: () {},
    onHighSpeed: (limit, insideCityArea) {},
    onSpeedLimit: (speed, limit, insideCityArea) {},
    onNormalSpeed: (limit, insideCityArea) {},
    onEnterDayMode: () {},
    onEnterNightMode: () {},
);

// Create alarm service based on the previously created listener
AlarmService alarmService = AlarmService(alarmListener);
```

Each callback method listed above can be specified to receive notifications about different events or topics, depending on your needs. These events cover a range of scenarios, such as boundary crossings, monitoring state changes, tunnel entries or exits, speed limits, and more. By customizing the callbacks, you can tailor the notifications to suit specific use cases, ensuring that the system responds appropriately to various triggers or conditions.

The AlarmListener and AlarmService objects must remain in memory for the duration of the notification period. If these objects are removed, the callbacks will not be triggered.
It's recommended to keep the ``AlarmListener`` and ``AlarmService`` variables in a class that is alive during the whole session.

## Change the alarm listener

The alarm listener associated with the alarm service can be updated at any time, allowing for dynamic configuration and flexibility in handling various notifications.
```dart
AlarmListener newAlarmListener = AlarmListener();
alarmService.alarmListener = newAlarmListener;
```

The callbacks within a ``AlarmListener`` can also be overridden at any time:
```dart
alarmListener.registerOnEnterDayMode(() {
    showSnackbar("Day mode entered");
});
```

Only one callback per event can be assigned to a listener. For instance, if a new onEnterDayMode callback is registered using ``registerOnEnterDayMode``, only the most recently set callback will be invoked when the event occurs.
