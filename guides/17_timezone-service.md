---
description: Documentation for Timezone Service
title: Timezone Service
---

# Timezone service

The `TimezoneService` provides functionality for managing and retrieving time zone information.
It allows you to transform a UTC `DateTime` to a specific time zone, but also provides other details regarding the offset and the timezone.

The `TimezoneService` class provides methods to get timezone information online (more accurate and takes into account new changes) or offline (faster, works regardless of network access but may be outdated).

- `getTimezoneInfoFromCoordinates` and `getTimezoneInfoFromTimezoneId` methods retrieve timezone information from an online service.

- `getTimezoneInfoFromCoordinatesSync` and `getTimezoneInfoFromTimezoneIdSync` methods provide similar functionality but use built-in timezone data, which may be outdated. A SDK update is required once in a while to refresh timezone data.

## The TimezoneResult structure

The `TimezoneResult` class represents the result of a time zone lookup operation. It contains the following properties:

| Member       | Type             | Description                                                                                            |
|--------------|------------------|--------------------------------------------------------------------------------------------------------|
| `dstOffset`  | `Duration`       | Daylight Saving Time (DST) offset.                                                                     |
| `utcOffset`  | `Duration`       | The raw UTC offset, excluding DST. Can be negative.                                                    |
| `offset`     | `Duration`       | The total offset in respect to UTC (`dstOffset` + `utcOffset`). Can be negative.                                         |
| `status`     | `TimeZoneStatus` | Status of the response. See the values below.                                                          |
| `timezoneId` | `String`         | The timezone identifier in format `Continent/City_Name`. Examples: `Europe/Paris`, `America/New_York`. |
| `localTime`  | `DateTime`       | The local time as a `UTC DateTime` object, but representing the local time of the requested timezone, affected by offsets. Use the `day`, `hour`, `minute`, `second` getters from the `DateTime` object. |

The `localTime` returned via the `TimezoneService` methods has the `isUtc` property set to `true`, but they represent the local time in the specified time zone.
This is a limitation of the `DateTime` class which allows only UTC or local time.

The `TimeZoneStatus` enum represents the status of a time zone lookup operation. It can have the following values:

- `success` - the request was successful.

- `invalidCoordinate` - the provided geographic coordinates were invalid or out of range.

- `wrongTimezoneId` - the provided timezone identifier was malformed or not recognized.

- `wrongTimestamp` - the provided timestamp (DateTime) was invalid or could not be parsed.

- `timezoneNotFound` - no timezone could be found for the given input.

- `successUsingObsoleteData` - the request succeeded but the service had to fall back to obsolete/outdated timezone data. Relevant when using `getTimezoneInfoFromCoordinatesSync` and `getTimezoneInfoTimezoneIdSync`. Please update the SDK.

## Get timezone info by coordinates

### Get timezone info by coordinates online

The `getTimezoneInfoFromCoordinates` method allows you to retrieve time zone information based on geographic coordinates (latitude and longitude) and an UTC `DateTime`.
This can be useful for applications that need to determine the local time zone for a specific location.
```dart
TimezoneService.getTimezoneInfoFromCoordinates(
    coords: Coordinates(latitude: 55.626, longitude: 37.457),
    time: DateTime.utc(2025, 7, 1, 6, 0),   // <-- This is always the Datetime in UTC
    onComplete: (GemError error, TimezoneResult? result) {
        if (error != GemError.success) {
            // The request failed, handle the error    
        } else {
            // Do something with the result
        }
    }
);
```

### Get timezone info by coordinates offline

The `getTimezoneInfoFromCoordinatesSync` method allows you to retrieve time zone information based on geographic coordinates (latitude and longitude) and an UTC `DateTime` without making an online request. This can be useful for applications that need to determine the local time zone for a specific location without relying on network access.
```dart
TimezoneResult? result = TimezoneService.getTimezoneInfoFromCoordinatesSync(
    coords: Coordinates(latitude: 55.626, longitude: 37.457),
    time: DateTime.utc(2025, 7, 1, 6, 0), // <-- Always in UTC
);
```

## Get timezone info by timezone ID

### Get timezone info by timezone ID online

The `getTimezoneInfoFromTimezoneId` method allows you to retrieve time zone information based on a specific timezone ID and an UTC `DateTime`.
```dart
TimezoneService.getTimezoneInfoFromTimezoneId(
    timezoneId: 'Europe/Moscow',
    time: DateTime.utc(2025, 7, 1, 6, 0), // <-- Always in UTC
    onComplete: (GemError error, TimezoneResult? result) {
        if (error != GemError.success) {
            // The request failed, handle the error
        } else {
            // Do something with the result
        }
    },
);
```

### Get timezone info by timezone ID offline

The `getTimezoneInfoFromTimezoneIdSync` method allows you to retrieve time zone information based on a specific timezone ID and an UTC `DateTime` without making an online request. This can be useful for applications that need to determine the local time zone for a specific location without relying on network access.
```dart
TimezoneResult? result = TimezoneService.getTimezoneInfoTimezoneIdSync(
    timezoneId: 'Europe/Moscow',
    time: DateTime.utc(2025, 7, 1, 6, 0), // <-- Always in UTC
);
```

