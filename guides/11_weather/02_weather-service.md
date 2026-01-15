---
description: Documentation for Weather Service
title: Weather Service
---

# Weather Service

The `WeatherService` class provides methods for retrieving current, hourly, and daily weather forecasts.

---

## Get Current Weather Forecast

Use the `getCurrent` method to retrieve the current weather forecast. Provide coordinates for the desired location, and the forecast will be returned through `onComplete`.
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

Verify that `LocationForecast` contains `Conditions` and each `Condition` includes a `Parameter`. If data is unavailable for the specified location and time, the API may return empty lists.

The result contains as many `LocationForecast` objects as coordinates provided to the `coords` parameter.

---

## Get Hourly Weather Forecast

Use the `getHourlyForecast` method to retrieve hourly weather forecasts. Specify the number of hours and coordinates for the desired location.
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

The number of requested hours must not exceed 240. Exceeding this limit results in an empty response and a `GemError.outOfRange` error.

---

## Get Daily Weather Forecast

Use the `getDailyForecast` method to retrieve daily weather forecasts. Specify the number of days and coordinates for the desired location.
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

The number of requested days must not exceed 10. Exceeding this limit results in an empty response and a `GemError.outOfRange` error.

---

## Get Weather Forecast with Duration

Use the `getForecast` method to retrieve weather forecasts for specific times and coordinates. Provide a list of `WeatherDurationCoordinates`, and `onComplete` returns as many `LocationForecast` objects as items in the list.
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

The `duration` parameter in `WeatherDurationCoordinates` specifies the time offset into the future for the forecast.

---

## Relevant example demonstrating weather related features

- [Weather Forecast](/examples/places-search/weather-forecast)
