---
description: Documentation for Weather Service
title: Weather Service
---

# Weather Service

The `WeatherService` class contains methods for getting the current, hourly and daily forecasts.

## Get Current Weather Forecast

To retrieve the current weather forecast, use the static `getCurrent` method of the `WeatherService` class. Provide the coordinates for the desired location, and the forecast will be returned through the `onComplete`. The following example demonstrates how to fetch the current forecast for Paris coordinates.
```dart
    final locationCoordinates = Coordinates(
      latitude: 48.864716,
      longitude: 2.349014,
    );
    final weatherCurrentCompleter = Completer<List<LocationForecast>>();

    WeatherService.getCurrent(
      coords: [locationCoordinates],
      onComplete: (err, result) async {
        weatherCurrentCompleter.complete(result);
      },
    );

    final currentForecast = await weatherCurrentCompleter.future;
    showSnackbar("Forecast lenght list: ${currentForecast.length}");
```

The API user is responsible for verifying whether the `LocationForecast` contains any `Conditions`, and whether each `Condition` includes a `Parameter`. If data is unavailable for the specified location and time, the API may return empty lists of`Condition`s or `Parameters`s.

The result will contain as many `LocationForecast` objects as the list size of given coordinates to `coords` parameter.

## Get Hourly Weather Forecast

To retrieve the hourly weather forecast, use the static `getHourlyForecast` method of the `WeatherService` class. Provide the coordinates for the desired location, and the forecast will be returned through the `onComplete`. The following example demonstrates how to fetch the hourly forecast for Paris coordinates.
```dart
  final locationCoordinates = Coordinates(
    latitude: 48.864716,
    longitude: 2.349014,
  );

  final weatherHourlyCompleter = Completer<List<LocationForecast>>();

  WeatherService.getHourlyForecast(
    hours: 24,
    coords: [locationCoordinates],
    onComplete: (err, result) async {
      weatherHourlyCompleter.complete(result);
    },
  );

  final hourlyForecast = await weatherHourlyCompleter.future;
  showSnackbar("Forecast lenght list: ${hourlyForecast.length}");
```

You'll need to provide the number of hours for which the forecast is requested.

The number of requested hours must not exceed 240. Exceeding this limit will result in an empty response and an error of type `GemError.outOfRange`.

## Get the Daily Forecast

To retrieve the daily weather forecast, use the static `getDailyForecast` method of the `WeatherService` class. Provide the coordinates for the desired location, and the forecast will be returned through the `onComplete`. The following example demonstrates how to fetch the daily forecast for Paris coordinates.
```dart
  final locationCoordinates = Coordinates(
    latitude: 48.864716,
    longitude: 2.349014,
  );

  final weatherDailyCompleter = Completer<List<LocationForecast>>();

  WeatherService.getDailyForecast(
    days: 10,
    coords: [locationCoordinates],
    onComplete: (err, result) async {
      weatherDailyCompleter.complete(result);
    },
  );

  final dailyForecast = await weatherDailyCompleter.future;
  showSnackbar("Forecast lenght list: ${dailyForecast.length}");
```

You'll need to provide the number of days for which the forecast is requested.

The number of requested days must not exceed 10. Exceeding this limit will result in an empty response and an error of type `GemError.outOfRange`.

## Get the Weather Forecast

To retrieve the weather forecast at whatever time and coordinates, use the static `getForecast` method of `WeatherService` class. You'll need to provide a list of `WeatherDurationCoordinates`, and the `onComplete` will retrieve as many `locationForecasts` as the size of `WeatherDurationCoordinates` list.
```dart
final weatherCompleter = Completer<List<LocationForecast>>();

WeatherService.getForecast(
    coords: [
      WeatherDurationCoordinates(
          coordinates: Coordinates(
            latitude: 48.864716,
            longitude: 2.349014,
          ),
          duration: Duration(days: 2))
    ],
    onComplete: (err, result) async {
      weatherCompleter.complete(result);
    });

final forecast = await weatherCompleter.future;

showSnackbar("Forecast lenght list: ${forecast.length}");
```

The `duration` parameter in `WeatherDurationCoordinates` specifies the time offset into the future for which the forecast is requested.

## Relevant example demonstrating weather related features

- [Weather Forecast](/examples/places-search/weather-forecast)
