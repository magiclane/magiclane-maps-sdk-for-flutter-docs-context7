---
description: Documentation for Settings Service
title: Settings Service
---

# Settings Service

The Settings Service stores key-value pairs in permanent storage using the `SettingsService` class. Settings are saved in a `.ini` file format.

---

## Step 1: Create a Settings Service

Create or open a settings storage using the factory constructor of `SettingsService`. If no path is provided, a default one is used:
```dart
final settings = SettingsService();
```

You can provide a custom path:
```dart
final settings = SettingsService(path: "/custom/settings/path");
```

Access the current file path where settings are stored:
```dart
final String currentPath = settings.path;
```

---

## Step 2: Add and get values

Store various types of data using set methods:
```dart
settings.setString("username", "john_doe");
settings.setBool("isLoggedIn", true);
settings.setInt("launchCount", 5);
settings.setLargeInt("highScore", 1234567890123);
settings.setDouble("volume", 0.75);
```

Retrieve values using get methods. These methods accept an optional `defaultValue` parameter returned when the key is not found in the selected group. The `defaultValue` does not set the value.
```dart
final String username = settings.getString("username", defaultValue: "guest");
final bool isLoggedIn = settings.getBool("isLoggedIn", defaultValue: false);
final int launchCount = settings.getInt("launchCount", defaultValue: 0);
final int highScore = settings.getLargeInt("highScore", defaultValue: 0);
final double volume = settings.getDouble("volume", defaultValue: 1.0);
```

When you set a value on one type and get it on another type, a conversion occurs:
```dart
settings.setInt("count", 1234);

String value = settings.getString("count"); // Returns '1234'
```

Each change may take up to one second to be written to storage. Use the `flush` method to ensure changes are written to permanent storage immediately.

---

## Step 3: Organize with groups

Groups organize settings into logical units. The default group is `DEFAULT`. Only one group can be active at a time, and nested groups are not allowed.
```dart
// All operations above this are made inside DEFAULT
settings.beginGroup("USER_PREFERENCES");

// All operations here are made inside USER_PREFERENCES

settings.beginGroup("OTHER_SETTINGS");

// All operations here are made inside OTHER_SETTINGS

settings.endGroup();
// All operations above this are made inside DEFAULT
```

Get the current group using the `group` getter.

The values passed to `beginGroup` are converted to upper-case.

A `flush` is automatically done after the group is changed.

---

## Step 4: Remove values

### Remove value by key

The `remove` method accepts a key (or pattern) and returns the number of deleted entries from the current group:
```dart
final int removedCount = settings.remove("username");
```

### Clear all values

Use the `clear` method to remove all settings from all groups:
```dart
settings.clear();
```

