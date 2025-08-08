---
description: Documentation for Other Alarms
title: Other Alarms
---

# Other alarms

The Maps SDK for Flutter provides advanced notification capabilities, enabling users to receive alerts for significant environmental changes. This includes notifications for entering and exiting tunnels, as well as transitions between day and night, based on the user’s current location and time of year.

## Get notified when entering and exiting tunnels

This snippet below demonstrates how to set up notifications for entering and exiting tunnels using the AlarmListener:
```dart
AlarmListener alarmListener = AlarmListener(
    onTunnelEntered: () {
        showSnackbar("Tunnel entered");
    },
    onTunnelLeft: () {
        showSnackbar("Tunnel left");
    },
);
```

## Get notified when transitioning between day and night

The Maps SDK for Flutter offers functionality to notify users when their location transitions between day and night, based on the geographical region and the corresponding seasonal changes:
```dart
AlarmListener alarmListener = AlarmListener(
    onEnterDayMode: () {
        showSnackbar("Day mode entered");
    },
    onEnterNightMode: () {
        showSnackbar("Night mode entered");
    },
);
```


