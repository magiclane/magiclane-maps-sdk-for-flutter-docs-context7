---
description: Documentation for Settings Service
title: Settings Service
---

# Settings Service

The Magic Lane SDK for Flutter provides functionality for storing key-value pairs in permanent storage. This is managed via the `SettingsService` class.

The settings are saved within a `.ini` file.

## Create a Settings Service

You can create or open a settings storage using the factory constructor of `SettingsService`. If no path is provided, a default one will be used:
```dart
final settings = SettingsService();
```

Or a custom path can be provided:
```dart
final settings = SettingsService(path: "/custom/settings/path");
```

You can also access the current file path where the settings are stored:
```dart
final String currentPath = settings.path;
```

## Add and get values

You can store various types of data using appropriate set methods:
```dart
settings.setString("username", "john_doe");
settings.setBool("isLoggedIn", true);
settings.setInt("launchCount", 5);
settings.setLargeInt("highScore", 1234567890123);
settings.setDouble("volume", 0.75);
```

To retrieve values, use the corresponding get methods. These methods take an optional `defaultValue` parameter which is returned when the key could not be found in the selected group.
The `defaultValue` does not set the value.
```dart
final String username = settings.getString("username", defaultValue: "guest");
final bool isLoggedIn = settings.getBool("isLoggedIn", defaultValue: false);
final int launchCount = settings.getInt("launchCount", defaultValue: 0);
final int highScore = settings.getLargeInt("highScore", defaultValue: 0);
final double volume = settings.getDouble("volume", defaultValue: 1.0);
```

When the set is made on a type and the get is made on another type, a conversion is done. For example:
```dart
settings.setInt("count", 1234);

String value = settings.getString("count"); // Returns '1234'
```

Each change may take up to one second to be written to storage. Use the `flush` method to ensure the changes are written to permanent storage.

## Groups

Groups allow you to organize settings in logical units. The default group is `DEFAULT`.
Only one group can be active at a time, and nested groups are not allowed.
```dart
// All operations above this are made inside DEFAULT
settings.beginGroup("USER_PREFERENCES");

// All operations here are made inside USER_PREFERENCES

settings.beginGroup("OTHER_SETTINGS");

// All operations here are made inside OTHER_SETTINGS

settings.endGroup();
// All operations above this are made inside DEFAULT
```

In order to get the current group use the `group` getter.

The values passed to `beginGroup` are converted to upper-case.

A `flush` is automatically done after the group is changed.

## Remove values

### Remove value by key

The `remove` method takes the key (or a pattern) and returns the number of deleted entries from the current group.
```dart
final int removedCount = settings.remove("username");
```

### Clear all values

Use the `clear` method which removes all the settings from all the groups.
