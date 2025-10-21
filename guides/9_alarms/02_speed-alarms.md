---
description: Documentation for Speed Alarms
title: Speed Alarms
---

# Speed warnings

The SDK provides features for monitoring and notifying users about speed limits and violations. You can configure alerts for when a user exceeds the speed limit, when the speed limit changes on the new road segment with respect to the previous, and when the user returns to a normal speed range (onNormalSpeed). The SDK also allows you to set customizable thresholds for speed violations, which can be adjusted for both city and non-city areas. These features help provide timely and relevant speed-related notifications based on the user's location and current speed.

## Configure the speed limit listeners
```dart
AlarmListener alarmListener = AlarmListener(
    onHighSpeed: (limit, insideCityArea) {
        if (insideCityArea) {
            showSnackbar("Speed limit exceeded while inside city area - limit is $limit m/s");
        } else {
            showSnackbar("Speed limit exceeded while outside city area - limit is $limit m/s");
        }
    },
    onSpeedLimit: (speed, limit, insideCityArea) {
        if (insideCityArea) {
            showSnackbar("New speed limit updated to $limit m/s while inside city area. The current speed is $speed m/s");
        } else {
            showSnackbar("New speed limit updated to $limit m/s while outside city area. The current speed is $speed m/s");
        }
    },
    onNormalSpeed: (limit, insideCityArea) {
        if (insideCityArea) {
            showSnackbar("Normal speed restored while inside city area - limit is $limit m/s");
        } else {
            showSnackbar("Normal speed restored while outside city area - limit is $limit m/s");
        }
    },
);

// Create alarm service based on the previously created listener
AlarmService alarmService = AlarmService(alarmListener);
```

The ``onHighSpeed`` will continuously send notifications while the user exceeds with a given threshold the maximum speed limit for the current road section.
The ``onSpeedLimit`` will be triggered once the current road section has a different speed than the previous road section.
The ``onNormalSpeed`` will be triggered once the user speed becomes within the limit of the maximum speed limit for the current road section.

Although the parameter is named ``insideCityArea``, it refers to areas with generally lower speed limits, such as cities, towns, villages, or similar settlements, regardless of their classification.

The `limit` parameter provided to `onSpeedLimit` will be 0 if the matched road section does not have a maximum speed limit available or if no road could be found.

## Set the threshold for speed

The threshold for the maximum speed excess that triggers the ``onHighSpeed`` callback can be configured as follows:
```dart
// Trigger onHighSpeed when the speed limit is exceeded by 1 m/s inside a city area
alarmService.setOverSpeedThreshold(threshold: 1, insideCityArea: true);    
// Trigger onHighSpeed when the speed limit is exceeded by 3 m/s inside a city area
alarmService.setOverSpeedThreshold(threshold: 3, insideCityArea: true);
```

## Get the threshold for the speed

The configured threshold can be accessed as follows:
```dart
double currentThresholdInsideCity = alarmService.getOverSpeedThreshold(true);
double currentThresholdOutsideCity = alarmService.getOverSpeedThreshold(false);
```

## Relevant examples demonstrating speed alarms related features

- [Speed TTS Warning](/examples/routing-navigation/speed-tts-warning)
