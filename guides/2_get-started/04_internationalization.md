---
description: Documentation for Internationalization
title: Internationalization
---

# Internationalization

The Magic Lane SDK for Flutter provides multilingual and localization support for global applications. Configure language settings, text-to-speech instructions, units of measurement, and number formats to match your users' preferences.

---

## Set the SDK language

Configure the SDK's language by selecting a language from the `SdkSettings.languageList` getter and assigning it using the `SdkSettings.language` setter.
```dart
Language? engLang = SdkSettings.getBestLanguageMatch("eng");
SdkSettings.language = engLang!;
```

The `languagecode` follows the [ISO 639-3 standard](https://iso639-3.sil.org/code_tables/639/data). Multiple variants may exist for a single language code. Filter further using the `regionCode` field within the `Language` object, which adheres to the [ISO 3166-1](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) standard.

By default, the SDK language is set to the device's preferred language.

**What's affected by the SDK language setting:**

- **Landmark search results** - displayed names of landmarks

- **Overlay items** - names and details shown on the map and in search results

- **Landmark categories** - displayed names

- **Overlays** - displayed names

- **Navigation and routing instructions** - text-based instructions for on-screen display (text-to-speech instructions remain unaffected)

- **`name` field of `GemParameter` objects** - returned by weather forecasts, overlay item previews, traffic event previews, content store parameters, route extra information, and data source parameters

- **Content store item names** - names of downloadable road maps and styles

- **Wikipedia external information** - titles, descriptions, and localized URLs

---

## Set text-to-speech language

The TTS (text-to-speech) instruction language is managed separately from the SDK language. You can change between a wide selection of voices.

The SDK language and TTS language are not automatically synchronized. Keep these settings in sync based on your use case.

See the [voice guidance guide](../navigation/voice-guidance) for more details.

---

## Configure map language

Set the map language to display location names consistently worldwide or in their native language.

### Use automatic translation
```dart
SdkSettings.mapLanguage = MapLanguage.automatic;
```

### Use native language

Display location names in their native language for the respective region:
```dart
SdkSettings.mapLanguage = MapLanguage.native;
```

**Example:**

When the SDK language is set to English:

- `MapLanguage.automatic` - displays "Beijing"

- `MapLanguage.native` - displays "北京市"

---

## Configure units of measurement

Change between different unit systems using the `unitSystem` property of the `SdkSettings` class.

**Supported unit systems:**

| Unit         | Distance               | Temperature |
|--------------|------------------------|-------------|
| `metric`     | Kilometers and meters  | Celsius     |
| `imperialUK` | Miles and yards        | Celsius     |
| `imperialUS` | Miles and feet         | Fahrenheit  |

**What's affected:**

- Distance values in navigation and routing instructions (display and TTS)

- Map scale values

- Temperature values in weather forecasts

All numeric getters and setters use SI (International System) units, regardless of the `unitSystem` setting:

- Distance - meters

- Time - seconds

- Mass - kilograms

- Speed - meters per second

- Power - watts

Exceptions to this convention are documented in the API reference and user guide. For example, the `TruckProfile` class uses centimeters for dimension properties.

---

## DateTime convention

Most members returning a `DateTime` value use the UTC time zone.

The following members return **local time** with the `isUtc` flag set to `true`:

- `PTStopTime.departureTime`

- `PTTrip.tripDate`

- `RoutingPreferences.timestamp`

These exceptions are documented in the API reference and user guide.

Use the `TimezoneService` class to convert between UTC and local time zones. See the [TimezoneService guide](../timezone-service) for more details.

---

## Configure number separators

Format numbers using custom characters for decimal and digit group separators. Configure these settings via the `SdkSettings` class.

### Set decimal separator

The **decimal separator** divides the whole and fractional parts of a number.

Use the `decimalSeparator` property of the `SdkSettings` class:
```dart
SdkSettings.decimalSeparator = ',';
```

### Set digit group separator

The **digit group separator** groups large numbers (e.g., separating thousands).

Use the `digitGroupSeparator` property of the `SdkSettings` class:
```dart
SdkSettings.digitGroupSeparator = '.';
```

---

## Convert ISO codes

Convert country or language codes between ISO formats using the `ISOCodeConversions` class.

- **Country codes** - use `ISOCodeConversions.convertCountryIso`

- **Language codes** - use `ISOCodeConversions.convertLanguageIso`
```dart
// Convert country ISO from ISO 3166-1 alpha-2 to alpha-3
final res1 = ISOCodeConversions.convertCountryIso("BR", ISOCodeType.iso_3); // BRA

// Convert country ISO from ISO 3166-1 alpha-3 to alpha-2
final res2 = ISOCodeConversions.convertCountryIso("BRA", ISOCodeType.iso_2); // BR

// Convert language ISO from ISO 639-3 to ISO 639-1
final res3 = ISOCodeConversions.convertLanguageIso("hun", ISOCodeType.iso_2); // hu

// Convert language ISO from ISO 639-1 to ISO 639-3
final res4 = ISOCodeConversions.convertLanguageIso("hu", ISOCodeType.iso_3); // hun
```


