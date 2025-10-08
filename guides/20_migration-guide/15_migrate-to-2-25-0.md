---
description: Documentation for Migrate To 2 25 0
title: Migrate To 2 25 0
---

# Migrate to 2.25.0

This guide outlines the breaking changes introduced in SDK version 2.25.0. Required updates may vary depending on your use case.

Additionally, new features and bugfixes have been introduced and are **not** documented here. For a comprehensive list of changes, please refer to the changelog.

This release introduces small additions and fixes important bugs.

## Removed *create* method from *AddressInfo*

The static `create` method has been removed from the `AddressInfo` class. To create a new instance, use the public constructor instead.

Before:
```dart
final address = AddressInfo.create();
```

After:
```dart
final address = AddressInfo();
```

This change simplifies the API and encourages direct use of the constructor, which is now the only way to instantiate `AddressInfo`.

## Changed type of *coordinates* property in *WGS84Projection*

The type of the `coordinates` property in the `WGS84Projection` class has changed from `Coordinates?` (nullable) to `Coordinates` (non-nullable).

Before:
```dart
WGS84Projection projection = ...
Coordinates? coords = projection.coordinates;
if (coords != null) {
	// Use coords
}
```

After:
```dart
WGS84Projection projection = ...
Coordinates coords = projection.coordinates;
if (coords.isValid){
    // Use coords
}
```

If your code checks for null, you can now safely remove those checks and replace them with a `.isValid` verification.

This change improves null safety and simplifies code that uses the `Projection` class.

## Added optional *language* parameter to *SdkSettings.setVoiceByPath*

The `setVoiceByPath` method of the `SdkSettings` class now accepts an optional `language` parameter.

Before:
```dart
SdkSettings.setVoiceByPath(path);
```

After:
```dart
Language language = ...
SdkSettings.setVoiceByPath(path, language: language);
```

No changes are required for existing code. If you want to specify a language for the voice, pass the `language` argument. Otherwise, the method behaves as before.
The `language` parameter is used only for computer voice, for better match between the voice instruction language and the voice.
