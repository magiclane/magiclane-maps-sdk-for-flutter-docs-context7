---
description: Documentation for Driver Behaviour
title: Driver Behaviour
---

# Driver Behaviour

The Driver Behaviour feature enables the analysis and scoring of a driver's behavior during a trip, identifying risky driving patterns and summarizing them with safety scores. This feature tracks both real-time and session-level driving events, such as harsh braking, cornering, or ignoring traffic signs, and evaluates overall risk using multiple criteria.

This data can be used to offer user feedback, identify unsafe habits, and assess safety levels over time. All information is processed using on-device sensor data (via the configured `DataSource`) and optionally matched to the road network if `useMapMatch` is enabled.

## Starting and Stopping Analysis

To use the Driver Behaviour module, a session must be started using the `startAnalysis` method of the `DriverBehaviour` object. The session is closed using `stopAnalysis`, which returns a `DriverBehaviourAnalysis` instance representing the complete analysis.
```dart
final driverBehaviour = DriverBehaviour(
  dataSource: myDataSource,
  useMapMatch: true,
);

bool started = driverBehaviour.startAnalysis();

// ... after some driving

DriverBehaviourAnalysis? result = driverBehaviour.stopAnalysis();

if (result == null){
  print("The analysis is invalid and cannot be used");
  return;
}
```

All `DriverBehaviourAnalysis` instances expose an `isValid` getter to determine whether the analysis is valid. Always verify this property before accessing or relying on the data it contains.

##  Inspecting a Driving Session

The result returned by stopAnalysis() (or via getLastAnalysis()) contains aggregate and detailed information on the trip:
```dart
if (result == null) {
  print("The analysis is invalid and cannot be used");
  return;
}

int startTime = result.startTime;
int finishTime = result.finishTime;
double distance = result.kilometersDriven;
double drivingDuration = result.minutesDriven;
double speedingTime = result.minutesSpeeding;
```

The session also includes risk scores:
```dart
DrivingScores? scores = result.drivingScores;
if (scores == null) {
  showSnackbar("No driving scores available");
  return;
}

double speedRisk = scores.speedAverageRiskScore;
double brakingRisk = scores.harshBrakingScore;
double fatigue = scores.fatigueScore;
double overallScore = scores.aggregateScore;
```

Each score ranges from 0 (unsafe) to 100 (safe). A score of -1 means invalid or unavailable.

##  Inspecting a Driving Session

Use the `drivingEvents` property of the session result to access discrete driving incidents that were detected:
```dart
List<MappedDrivingEvent> events = result.drivingEvents;
for (final event in events) {
  print("Event at ${event.latitudeDeg}, ${event.longitudeDeg} at ${event.time} with type ${event.eventType}");
}
```

Event types are defined by the DrivingEvent enum:

## Driving Event Types

| Enum Value            | Description                     |
|-----------------------|---------------------------------|
| `noEvent`             | No event                        |
| `startingTrip`        | Starting a trip                 |
| `finishingTrip`       | Finishing a trip                |
| `resting`             | Resting                         |
| `harshAcceleration`   | Harsh acceleration              |
| `harshBraking`        | Harsh braking                   |
| `cornering`           | Cornering                       |
| `swerving`            | Swerving                        |
| `tailgating`          | Tailgating                      |
| `ignoringSigns`       | Ignoring traffic signs          |

## Real-time Feedback

If the analysis is ongoing, you can fetch real-time scores using:
```dart
DrivingScores? instantScores = driverBehaviour.getInstantaneousScores();
```

These reflect the user's current behavior and are useful for immediate in-app feedback.

## Stop Analysis and Get Last Analysis

To stop an ongoing analysis you can use:
```dart
DriverBehaviourAnalysis? analysis = driverBehaviour.stopAnalysis();
if (analysis == null) {
  print("No valid analysis available");
  return;
}
```

You can also retrieve the last completed analysis:
```dart
DriverBehaviourAnalysis? lastAnalysis = driverBehaviour.getLastAnalysis();
if (lastAnalysis == null) {
  print("No valid analysis available");
  return;
}
```

## Retrieve Past Analyses

All completed sessions are stored locally and accessible via:
```dart
List<DriverBehaviourAnalysis> pastSessions = driverBehaviour.getAllDriverBehaviourAnalyses();
```

You can also obtain a combined analysis over a time interval:
```dart
DateTime start = DateTime.now().subtract(Duration(days: 7));
DateTime end = DateTime.now();

DriverBehaviourAnalysis? combined = driverBehaviour.getCombinedAnalysis(start, end);
```

## Data Cleanup

To save space or comply with privacy policies, older sessions can be erased:
```dart
driverBehaviour.eraseAnalysesOlderThan(DateTime.now().subtract(Duration(days: 30)));
```

Driver behaviour analysis requires a properly configured DataSource. See the [Positioning guide](positioning/get-started-positioning) to set up your data pipeline. To ensure reliable results, make sure to start and stop the analysis appropriately and avoid frequent interruptions or overlapping sessions.

## Relevant example demonstrating driver behavior-related features

- [Driver Behaviour](/docs/flutter/examples/routing-navigation/driver-behaviour)
