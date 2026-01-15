---
description: Documentation for Other Alarms
title: Other Alarms
---

# Other alarms

Receive alerts for environmental changes such as entering or exiting tunnels and transitions between day and night.

---

## Get notified for tunnel events

Set up notifications for entering and exiting tunnels using the `AlarmListener`:
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

---

## Get notified for day and night transitions

Receive notifications when your location transitions between day and night based on geographical region and seasonal changes:
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


