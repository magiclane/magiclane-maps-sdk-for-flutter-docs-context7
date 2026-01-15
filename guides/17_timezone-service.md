---
description: Documentation for Timezone Service
title: Timezone Service
---

# Timezone service

The `TimezoneService` provides functionality for managing and retrieving time zone information based on coordinates or timezone identifiers.

The service offers both online and offline methods. Online methods are more accurate and reflect recent timezone changes, while offline methods are faster and work without network access but may contain outdated data.

---

## Available methods

The `TimezoneService` class provides four main methods:

- `getTimezoneInfoFromCoordinates` - retrieves timezone information from coordinates using an online service

- `getTimezoneInfoFromTimezoneId` - retrieves timezone information from a timezone ID using an online service

- `getTimezoneInfoFromCoordinatesSync` - retrieves timezone information from coordinates using built-in data (offline)

- `getTimezoneInfoFromTimezoneIdSync` - retrieves timezone information from a timezone ID using built-in data (offline)

Offline methods use built-in timezone data that may become outdated. Update the SDK regularly to refresh timezone data.

---

## Understand the TimezoneResult structure

The `TimezoneResult` class represents the result of a timezone lookup operation and contains the following properties:

| Member       | Type             | Description                                                                                            |
|--------------|------------------|--------------------------------------------------------------------------------------------------------|
| `dstOffset`  | `Duration`       | Daylight Saving Time (DST) offset                                                                      |
| `utcOffset`  | `Duration`       | Raw UTC offset, excluding DST - can be negative                                                        |
| `offset`     | `Duration`       | Total offset relative to UTC (`dstOffset` + `utcOffset`) - can be negative                            |
| `status`     | `TimeZoneStatus` | Status of the response - see values below                                                              |
| `timezoneId` | `String`         | Timezone identifier in format `Continent/City_Name` (e.g., `Europe/Paris`, `America/New_York`)        |
| `localTime`  | `DateTime`       | Local time as a `UTC DateTime` object representing the local time of the requested timezone           |

The `localTime` property has `isUtc` set to `true` but represents local time in the specified timezone. This is a `DateTime` class limitation that only allows UTC or local time representation.

### TimeZoneStatus values

The `TimeZoneStatus` enum indicates the result of a timezone lookup operation:

- `success` - request completed successfully

- `invalidCoordinate` - provided coordinates were invalid or out of range

- `wrongTimezoneId` - provided timezone identifier was malformed or not recognized

- `wrongTimestamp` - provided timestamp was invalid or could not be parsed

- `timezoneNotFound` - no timezone found for the given input

- `successUsingObsoleteData` - request succeeded using outdated timezone data (update the SDK)

---

## Retrieve timezone by coordinates

### Online lookup

Use `getTimezoneInfoFromCoordinates` to retrieve timezone information based on geographic coordinates and a UTC `DateTime`:
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

### Offline lookup

Use `getTimezoneInfoFromCoordinatesSync` to retrieve timezone information from built-in data without network access:
```dart
TimezoneResult? result = TimezoneService.getTimezoneInfoFromCoordinatesSync(
    coords: Coordinates(latitude: 55.626, longitude: 37.457),
    time: DateTime.utc(2025, 7, 1, 6, 0), // <-- Always in UTC
);
```

---

## Retrieve timezone by timezone ID

### Online lookup

Use `getTimezoneInfoFromTimezoneId` to retrieve timezone information based on a timezone identifier and a UTC `DateTime`:
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

### Offline lookup

Use `getTimezoneInfoFromTimezoneIdSync` to retrieve timezone information from built-in data without network access:
```dart
TimezoneResult? result = TimezoneService.getTimezoneInfoFromTimezoneIdSync(
    timezoneId: 'Europe/Moscow',
    time: DateTime.utc(2025, 7, 1, 6, 0), // <-- Always in UTC
);
```

