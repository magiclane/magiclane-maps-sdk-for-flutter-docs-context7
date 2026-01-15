---
description: Documentation for Base Entities
title: Base Entities
---

# Base Entities

Weather-related functionalities are organized into distinct classes, each designed to encapsulate specific weather data.

The main classes include `LocationForecast`, `Conditions`, and `Parameter`. This guide provides a detailed explanation of each class and its purpose.

---

## LocationForecast

The `LocationForecast` class retains data such as the forecast update datetime, the geographic location, and forecast data.

| Property | Type | Description |
|----------|------|-------------|
| `updated` | `DateTime` | Forecast update datetime (UTC) |
| `coord` | `Coordinates` | Geographic location |
| `forecast` | `List<Conditions>` | Forecast data |

---

## Conditions

The `Conditions` class retains weather conditions for a given timestamp.

| Property | Type | Description |
|----------|------|-------------|
| `type` | `String` | For possible values see  add ref[PredefinedParameterTypeValues] |
| `stamp` | `DateTime` | Datetime for condition (UTC) |
| `image` | `Uint8List` | Image representation as `Uint8List` |
| `img` | `Img` | The conditions image as `Img` |
| `description` | `String` | Description translated according to the current SDK language |
| `daylight` | `Daylight` | Daylight condition |
| `params` | `List<Parameter>` | Parameter list |

### PredefinedParameterTypeValues

The `PredefinedParameterTypeValues` class contains the common values for `Parameter.type` and `Conditions.type`.

| Property | Description | Unit |
|----------|-------------|------|
| `airQuality` | 'AirQuality' |  -  |
| `dewPoint` | 'DewPoint' |  °C  |
| `feelsLike` | 'FeelsLike' |  °C  |
| `humidity` | 'Humidity' |   %  |
| `pressure` | 'Pressure' |  mb |
| `sunRise` | 'Sunrise' |  -  |
| `sunSet` | 'Sunset' |  -  |
| `temperature` | 'Temperature' |  °C  |
| `uv` | 'UV' |  -  |
| `visibility` | 'Visibility' |  km |
| `windDirection` | 'WindDirection' | ° |
| `windSpeed` | 'WindSpeed' | km/h |
| `temperatureLow` | 'TemperatureLow' |  °C  |
| `temperatureHigh` | 'TemperatureHigh' |  °C  |

The `WeatherService` may return data with varying property types, depending on data availability. A response might include only a subset of the values listed above.

---

## Parameter

The `Parameter` class contains weather parameter data.

| Property | Type | Description |
|----------|------|-------------|
| `type` | `String` | For possible values see add ref[PredefinedParameterTypeValues] |
| `value` | `double` | Value |
| `name` | `String` | Name translated according to the current SDK language |
| `unit` | `String` | Unit |

---

## Related examples

- [Weather Forecast](/examples/places-search/weather-forecast)
