---
description: Documentation for Route Preferences
title: Route Preferences
---

# Handling RoutePreferences

Before computing a route, we need to specify some route options. 

The most generic supported route options are briefly presented in the following table.

| Preference                          | Explanation                                            | Default Value                               |
|-------------------------------------|--------------------------------------------------------|---------------------------------------------|
| accurateTrackMatch                  | Enables accurate track matching for routes.            | true                                        |
| allowOnlineCalculation              | Allows online calculations.                            | true                                        |
| alternativeRoutesBalancedSorting    | Balances sorting of alternative routes.                | true                                        |
| alternativesSchema                  | Defines the schema for alternative routes.             | RouteAlternativesSchema.defaultSchema       |
| automaticTimestamp                  | Automatically includes a timestamp.                    | true                                        |
| departureHeading                    | Sets departure heading and accuracy.                   | DepartureHeading(heading: -1, accuracy: 0)  |
| ignoreRestrictionsOverTrack         | Ignores restrictions over the track.                   | false                                       |
| maximumDistanceConstraint           | Enables maximum distance constraints.                  | true                                        |
| pathAlgorithm                       | Algorithm used for path calculation.                   | RoutePathAlgorithm.ml                       |
| pathAlgorithmFlavor                 | Flavor for the path algorithm.                         | RoutePathAlgorithmFlavor.magicLane          |
| resultDetails                       | Level of details in the route result.                  | RouteResultDetails.full                     |
| routeRanges                         | Ranges for the routes.                                 | []                                          |
| routeRangesQuality                  | Quality level for route ranges.                        | 100                                         |
| routeType                           | Preferred route type.                                  | RouteType.fastest                           |
| timestamp                           | Custom timestamp for the route. Used with PT Routes to specify the desired arrival/departure time. It represents the **local time** of the route start/end, but with the `isUtc` flag set to `true` | null (current time will be used) |
| transportMode                       | Transport mode for the route.                          | RouteTransportMode.car                      |

In order to compute the `timestamp` in the required format, please check the snippet below:
```dart
final departureLandmark = Landmark.withLatLng(latitude: 45.65, longitude: 25.60);
final destinationLandmark = Landmark.withLatLng(latitude: 46.76, longitude: 23.58);

TimezoneService.getTimezoneInfoFromCoordinates(
  coords: departureLandmark.coordinates,
  time: DateTime.now().add(Duration(hours: 1)), // Compute time for one hour later
  onComplete: (error, result) {
    if (error != GemError.success) {
      // Handle error
      return;
    } else {
      final timestamp = result!.localTime;
      // Pass the timestamp to RoutePreferences
    }
  },
);
```

Check the [Timezone Service guide](../timezone-service) for more details.

Enabling complex structures creation options are presented in the following table:

| Preference                          | Explanation                                            | Default Value                               |
|-------------------------------------|--------------------------------------------------------|---------------------------------------------|
| buildConnections                    | Enables building of route connections.                 | false                                       |
| buildTerrainProfile                 | Enables building of terrain profile.                   | BuildTerrainProfile(enable: false)          |

Route specific options for custom profiles are presented in the following table:

| Preference                          | Explanation                                            | Default Value                               |
|-------------------------------------|--------------------------------------------------------|---------------------------------------------|
| bikeProfile                         | Profile configuration for bikes.                       | null                                        |
| carProfile                          | Profile configuration for cars.                        | null                                        |
| evProfile                           | Profile configuration for electric vehicles.           | null                                        |
| pedestrianProfile                   | Profile configuration for pedestrians.                 | PedestrianProfile.walk                      |
| truckProfile                        | Profile configuration for trucks.                      | null                                        |

Avoid options are presented in the following table:

| Preference                          | Explanation                                            | Default Value                               |
|-------------------------------------|--------------------------------------------------------|---------------------------------------------|
| avoidBikingHillFactor               | Factor to avoid biking hills.                          | 0.5                                         |
| avoidCarpoolLanes                   | Avoids carpool lanes.                                  | false                                       |
| avoidFerries                        | Avoids ferries in the route.                           | false                                       |
| avoidMotorways                      | Avoids motorways in the route.                         | false                                       |
| avoidTollRoads                      | Avoids toll roads in the route.                        | false                                       |
| avoidTraffic                        | Strategy for avoiding traffic.                         | TrafficAvoidance.none                       |
| avoidTurnAroundInstruction          | Avoids turn-around instructions.                       | false                                       |
| avoidUnpavedRoads                   | Avoids unpaved roads in the route.                     | false                                       |

Emergency vehicles preferences are shown in the following table:

| Preference                          | Explanation                                            | Default Value                               |
|-------------------------------------|--------------------------------------------------------|---------------------------------------------|
| emergencyVehicleExtraFreedomLevels  | Extra freedom levels for emergency vehicles.           | 0                                           |
| emergencyVehicleMode                | Enables emergency vehicle mode.                        | false                                       |

Public Transport preferences are shown in the following table:
| Preference                          | Explanation                                            | Default Value                               |
|-------------------------------------|--------------------------------------------------------|---------------------------------------------|
| algorithmType                       | Algorithm type used for routing.                       | PTAlgorithmType.departure                   |
| minimumTransferTimeInMinutes        | Minimum transfer time in minutes.                      | 1                                           |
| maximumTransferTimeInMinutes        | Sets maximum transfer time in minutes.                 | 300                                         |
| maximumWalkDistance                 | Maximum walking distance in meters.                    | 5000                                        |
| sortingStrategy                     | Strategy for sorting routes.                           | PTSortingStrategy.bestTime                  |
| routeTypePreferences                | Preferences for route types.                           | RouteTypePreferences.none                   |
| useBikes                            | Enables use of bikes in the route.                     | false                                       |
| useWheelchair                       | Enables wheelchair-friendly routes.                    | false                                       |
| routeGroupIdsEarlierLater           | IDs for earlier/later route groups.                    | []                                          |

A short example of how they can be used to compute the fastest car route and also to compute a terrain profile is presented below:
```dart
final routePreferences = RoutePreferences(
    transportMode: RouteTransportMode.car,
    routeType: RouteType.fastest,
    buildTerrainProfile: BuildTerrainProfile(enable: true));
```

There are also properties that can't be set and only can be obtained for a route, in order to know how that route was computed:

| Preference                          | Explanation                                            | Default Value                               |
|-------------------------------------|--------------------------------------------------------|---------------------------------------------|
| routeResultType                     | Type of route result.                                  | RouteResultType.path                        |

## Computing truck routes

To compute routes for trucks we can write code like the following by initializing the `truckProfile` field of `RoutePreferences`:
```dart
// Define the departure.
final departureLandmark =
    Landmark.withLatLng(latitude: 48.87126, longitude: 2.33787);

// Define the destination.
final destinationLandmark =
    Landmark.withLatLng(latitude: 51.4739, longitude: -0.0302);

//highlight-start
final truckProfile = TruckProfile(
  height: 180, // cm
  length: 500, // cm
  width: 200, // cm
  axleLoad: 1500, // kg
  maxSpeed: 60, // km/h
  mass: 3000, // kg
  fuel: FuelType.diesel
);

// Define the route preferences with current truck profile and lorry transport mode.
final routePreferences = RoutePreferences(
  truckProfile: truckProfile,
  transportMode: RouteTransportMode.lorry, // <- This field is crucial
);
//highlight-end

TaskHandler? taskHandler = RoutingService.calculateRoute(
  [departureLandmark, destinationLandmark], routePreferences,
    (err, routes) {
      // handle results
});
```

`FuelType` can have the following values: petrol, diesel, lpg (liquid petroleum gas), electric. 

By default, all field except `fuel` have default value 0, meaning they are not considered in the routing. `fuel` by default is `FuelType.diesel`.

## Computing caravan routes

Certain vehicles, such as caravans or trailers, may be restricted on some roads due to their size or weight, yet still permitted on roads where trucks are prohibited.

To calculate routes for caravans or trailers, we can use the `truckProfile` field of `RoutePreferences` with the appropriate dimensions and weight.
```dart
// Define the departure.
final departureLandmark =
    Landmark.withLatLng(latitude: 48.87126, longitude: 2.33787);

// Define the destination.
final destinationLandmark =
    Landmark.withLatLng(latitude: 51.4739, longitude: -0.0302);

//highlight-start
final truckProfile = TruckProfile(
  height: 180, // cm
  length: 500, // cm
  width: 200, // cm
  axleLoad: 1500, // kg
);

// Define the route preferences with current truck profile and car transport mode.
final routePreferences = RoutePreferences(
  truckProfile: truckProfile,
  transportMode: RouteTransportMode.car, // <- This field is crucial to distinguish caravan from truck
);
//highlight-end

TaskHandler? taskHandler = RoutingService.calculateRoute(
  [departureLandmark, destinationLandmark], routePreferences,
    (err, routes) {
      // handle results
});
```

At least one of the fields `height`, `length`, `width` or `axleLoad` must be set to a non-zero value in order for the settings to be taken into account during routing. If all these fields are set to 0 then a normal car route will be calculated.
