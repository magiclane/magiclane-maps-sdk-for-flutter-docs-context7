---
description: Documentation for Internationalization
title: Internationalization
---

# Internationalization

The Magic Lane SDK for Flutter provides extensive multilingual and localization support, ensuring seamless integration for global applications.
By supporting a wide range of localizations, the SDK helps applications meet internationalization standards, enhance user engagement, and reduce friction for end-users across different markets.

## Set the SDK language

To configure the SDK's language, select a language from the ``SdkSettings.languageList`` getter and assign it using the ``SdkSettings.language`` setter.
```dart
Language? engLang = SdkSettings.languageList.where((lang) => lang.languagecode == 'eng').first;
SdkSettings.language = engLang;
```

The `languagecode` follows the [ISO 639-3 standard](https://iso639-3.sil.org/code_tables/639/data). Multiple variants may exist for a single language code. Further filtering can be applied using the `regionCode` field within the `Language` object, which adheres to the [ISO 3166-1](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) standard.

This operation modifies affects the following:

- **Landmark search results** : the displayed names of landmarks.

- **Overlay items** : names and details shown on the map and in search results.

- **Landmark category** : the displayed names.

- **Overlay** : the displayed names.

- **Navigation and routing instructions** : text-based instructions intended for on-screen display.  

  *(Text-to-speech instructions remain unaffected.)*

- **`name` field of `GemParameter` objects** returned by operations such as:

  - Weather forecasts

  - Overlay item previews

  - Traffic event previews

  - Content store parameters

  - Route extra information

  - Data source parameters

- **Content store item names** : names of downloadable road maps and styles.

- **Wikipedia external information** : titles, descriptions, and localized URLs.

## Set the text to speech instructions language

The language of the TTS instructions is different from the SDK language and it can be set using the 
```SdkSettings.setTTSLanguage```

 method. More details can be found inside the [voice guidance guide](../navigation/voice-guidance#change-the-text-to-speech-instructions-language).

The SDK Language and the TTS language are not automatically sincronized. It is the API user's responsability to keep these settings in sync, depending on the usecase.

## Map language

To ensure the map uses the same language all over the world, use the following code:
```dart
SdkSettings.mapLanguage = MapLanguage.automatic;
```

To show location names in their native language (where available) for the respective region, use this code:
```dart
SdkSettings.mapLanguage = MapLanguage.native;
```

For example, if the SDK language is set to English:

- When MapLanguage is set to automatic, landmarks like Beijing are translated and displayed on the map as “Beijing.”

- When MapLanguage is set to native, landmarks remain in their local language and are displayed as “北京市.”

## Set the unit of measurement

In order to change between the different unit systems use the `unitSystem` property of the `SdkSettings` class.

The following unit systems are supported:

| Unit         | Distance               | Temperature       |
|--------------|------------------------|-------------------|
| `metric`     | Kilometers and meters  | Celsius degrees   |
| `imperialUK` | Miles and yards        | Celsius degrees   |
| `imperialUS` | Miles and feet         | Farenheit degrees |

This change will affect:

- The values for distance interpolated in strings (both intended for display and TTS) such as navigation and routing instructions

- The values provided by the map scale

- The values for temperature used in weather forecasts

Keep in mind that all values returned by numeric getters and required by numeric setters are expressed using SI (International System) units, regardless of the `unitSystem` setting:

- meters for distance

- seconds for time

- kilogrames for mass

- meters per second for speed

- watts for power

Exceptions to this convention are explicitly documented in the API reference and user guide.
For example, the `TruckProfile` class uses centimeters for its dimension properties instead of SI units.

## Number separators

Numbers can be formatted using custom characters for decimal and digit group separators.  
These settings are controlled via the `SdkSettings` class.

### Set the decimal separator

The **decimal separator** divides the whole and fractional parts of a number.  

Use the `decimalSeparator` property of the `SdkSettings` class to set a character used as a decimal separator.

### Set the digit group separator 

The digit group separator is used to group large numbers (e.g., separating thousands).

Use the `digitGroupSeparator` property of the `SdkSettings` class to set a character used as a digit group separator.

## ISO Code Conversions

Various methods require and provide different standards of ISO codes. 
Use the `ISOCodeConversions` class to convert country or language codes between formats.

For country codes, use the static `ISOCodeConversions.convertCountryIso` method.
For language codes, use `ISOCodeConversions.convertLanguageIso` method.

For example:
```dart
// Converting conutry ISO from ISO 3166-1 alpha-2 to ISO 3166-1 alpha-3
final res1 = ISOCodeConversions.convertCountryIso("BR", ISOCodeType.iso_3); // BRA

// Converting conutry ISO from ISO 3166-1 alpha-3 to ISO 3166-1 alpha-2
final res2 = ISOCodeConversions.convertCountryIso("BRA", ISOCodeType.iso_2); // BR

// Converting language ISO from ISO 3166-1 alpha-3 to ISO 3166-1 alpha-2
final res3 = ISOCodeConversions.convertLanguageIso("hun", ISOCodeType.iso_2); // hu

// Converting language ISO from ISO 3166-1 alpha-2 to ISO 3166-1 alpha-3
final res4 = ISOCodeConversions.convertLanguageIso("hu", ISOCodeType.iso_3); // hun
```


