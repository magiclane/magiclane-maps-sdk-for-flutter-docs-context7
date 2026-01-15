---
description: Documentation for Get Started Alarms
title: Get Started Alarms
---

# Get started with Alarms

This guide explains how to set up and configure the alarm system to monitor geographic areas, routes, and receive notifications for various events.

---

The alarm system offers monitoring and notification functionalities for different alarm types, such as boundary crossings, speed limit violations, and landmark alerts. You can configure parameters like alarm distance, overspeed thresholds, and whether monitoring should occur even without a route being followed.

The system monitors specific geographic areas or routes and triggers alarms when predefined conditions are met, such as crossing a boundary or entering/exiting a tunnel. It provides customization options for alarm behavior based on location (e.g., inside or outside city limits). You can set up callbacks to receive notifications about specific events, including monitoring state changes or when a landmark alarm is triggered.

The system supports interaction with various alarm types, including overlay item and landmark alarms, and offers an easy interface for both setting and getting alarm-related information.

Multiple alarm services and listeners can operate simultaneously, allowing you to monitor various events concurrently.

---

## Configure alarm service and listener

To define an `AlarmListener` and create an `AlarmService`:
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

Each callback method can be specified to receive notifications about different events, such as boundary crossings, monitoring state changes, tunnel entries or exits, and speed limits. By customizing the callbacks, you can tailor the notifications to suit specific use cases.

The `AlarmListener` and `AlarmService` objects must remain in memory for the duration of the notification period. If these objects are removed, the callbacks will not be triggered. Keep the `AlarmListener` and `AlarmService` variables in a class that is alive during the whole session.

---

## Change alarm listener

The alarm listener associated with the alarm service can be updated at any time, allowing for dynamic configuration and flexibility in handling various notifications.
```dart
AlarmListener newAlarmListener = AlarmListener();
alarmService.alarmListener = newAlarmListener;
```

The callbacks within an `AlarmListener` can also be overridden at any time:
```dart
alarmListener.registerOnEnterDayMode(() {
    showSnackbar("Day mode entered");
});
```

Only one callback per event can be assigned to a listener. If a new `onEnterDayMode` callback is registered using `registerOnEnterDayMode`, only the most recently set callback will be invoked when the event occurs.

---

## Relevant examples demonstrating alarms related features

- [Areas Alarm](/examples/routing-navigation/areas-alarms)

- [Route Alarm](/examples/routing-navigation/route-alarms)
