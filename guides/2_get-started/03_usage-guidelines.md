---
description: Documentation for Usage Guidelines
title: Usage Guidelines
---

# Usage guidelines

This guide outlines best practices and important guidelines for using the Maps SDK for Flutter. Following these recommendations ensures code reliability and helps you avoid common pitfalls.

---

## SDK Class Usage

### Do not extend SDK classes

The SDK provides all necessary functionality out of the box. **Do not extend** any Maps SDK for Flutter classes. Instead, use the callback methods and interfaces provided by the SDK.

### Avoid @internal members

Members, methods, and fields annotated with `@internal` are for internal use only and should not be accessed by your application.

**Do not use:**

- **Fields:** `pointerId`, `mapId` getters/fields

- **Methods:**

  - Constructors initializing `pointerId` or `mapId` with `-1`

  - `fromJson` and `toJson` methods (JSON structure may change between versions)

  - Listener methods like `notify...` or `handleEvent`

  - Class `init` methods

- **FFI classes:** `GemKitPlatform`, `GemSdkNative`, `GemAutoreleaseObject`

Using internal elements can cause unexpected behavior and compatibility issues. These are not part of the public API and may change without notice.

---

## Working with Parameters

### DateTime requirements

Ensure `DateTime` values passed to SDK methods are valid positive UTC values. Values below `DateTime.utc(0)` are not supported.

### Units of measurement

The SDK uses SI units by default:

- **Distance:** Meters

- **Time:** Seconds

- **Mass:** Kilograms

- **Speed:** Meters per second

- **Power:** Watts

Configure the unit system for TTS instructions via `SDKSettings.unitSystem`.

Some fields use different units where more appropriate. For example, `TruckProfile` dimensions (`height`, `length`, `width`) are in **centimeters**. Check the API reference for specifics.

---

## Event Listeners

Follow these principles when working with event listeners:

- **Subscribe:** Use provided register methods or constructor parameters

- **Single callback:** Only one callback is active per event type; registering a new callback **overrides** the previous one

- **Unsubscribe:** Register an empty callback to unsubscribe

---

## Error Handling

### Check error codes

Always check error codes returned by SDK methods to handle failures appropriately.

The `GemError` enum indicates operation outcomes. Success values include:

- `success` — Operation completed successfully

- `reducedResult` — Partial results returned

- `scheduled` — Operation scheduled for later execution

### Using ApiErrorService

Some methods don't return `GemError` directly but may still fail. Use the `ApiErrorService` class to check the error code:
```dart
// Any SDK call...
GemError error = ApiErrorService.apiError;
if (error == GemError.success) print("The last operation succeeded.");
else print("The last operation failed with error: $error");
```

Check the error code using `ApiErrorService.apiError` **immediately after** the operation completes. Other SDK operations may overwrite the error code.

### Monitor error updates

Register a listener to get notified when error codes change:
```dart
ApiErrorService.registerOnErrorUpdate((error) {
    if (error == GemError.success) print("An operation succeeded");
    else print("An operation failed with error code $error")
});
```

---

## Asynchronous Operations

### Converting callbacks to Futures

Many SDK operations (search, routing, etc.) use callbacks to return results. Use a `Completer` to convert callbacks into `Future` objects:
```dart
Future<ResultType?> myFunction() async{
    final Completer<ResultType?> completer = Completer<ResultType?>();

    SomeGemService.doSomething(
        onComplete: (error, result){
            if (error == GemError.success && result != null){
                completer.complete(result);
            }
        }
    )

    return completer.future;
}
```

This simplifies asynchronous code handling. Learn more in the [Dart Completer documentation](https://api.flutter.dev/flutter/dart-async/Completer-class.html).

### Avoid isolates

Do not execute SDK operations inside isolates. The SDK already uses multithreading internally where appropriate. Using isolates may cause exceptions.

---

## Debugging and Logging

When reporting issues, always include SDK logs to help diagnose problems.

### Enable Dart-level logs

Configure logging to print messages to the console:
```dart
Debug.logCreateObject = true;
Debug.logCallObjectMethod = true;
Debug.logListenerMethod = true;

Debug.logLevel = GemLoggingLevel.all;
```

This enables logging for Dart-level operations.

### Enable native-level logs

The SDK also generates native (C++) logs, written to a file automatically.

**Add a log entry:**
```dart
Debug.log(level: GemDumpSdkLevel.info, message: "This is a log message");
```

**Set log level:**
```dart
await Debug.setSdkDumpLevel(GemDumpSdkLevel.verbose);
```

**Get log file path:**
```dart
String logFilePath = await Debug.sdkLogDumpPath;
```

**Platform support:**

- **Android:** All `GemLoggingLevel` values supported

- **iOS:** Only `GemDumpSdkLevel.silent` and `GemDumpSdkLevel.verbose` supported

Provide **both** Dart and native logs when reporting issues.

Logs may contain sensitive information. Review before sharing publicly.

### Information checklist for bug reports

When reporting issues, include:

- **Description:** Clear, concise bug description

- **Reproduction steps:** Include minimal code sample if possible

- **Expected vs. actual behavior**

- **Visual aids:** Screenshots or videos (if applicable)

- **SDK version**

- **Platform details:** iOS/Android, OS version, device model

- **Logs:** Console output during the issue

- **Location:** Geographic location for routing/navigation/search issues

- **SDK version:** Ensure you're using the latest version

---

## Resource Management

### Modifying resources safely

Some resources load before `GemKit.initialize()` to improve startup speed. Manually changing resources (maps, styles, icons) can cause crashes or be ignored, especially on Android.

**If you need to modify resources, use one of these approaches:**

#### Option 1: Disable early initialization

In `android/build.gradle`, add this to the `buildTypes` block:
```gradle
    buildTypes {
        debug {
            buildConfigField "boolean", "USE_EARLY_INIT", "false"
        }
        release {
            buildConfigField "boolean", "USE_EARLY_INIT", "false"
        }
        profile {
           buildConfigField "boolean", "USE_EARLY_INIT", "false"
        }
    }
```

#### Option 2: Initialize and release SDK

Unload resources before making changes:
```dart
await GemKit.initialize(...);
await GemKit.release();

// Modify the resources

await GemKit.initialize(...);
await GemKit.release();
```

This ensures resources are properly unloaded before applying changes.

---

## Common Issues

### Avoid name conflicts

The SDK's `Route` class may conflict with Flutter's `Route` class. Hide Flutter's version when importing:
```dart
import 'package:flutter/material.dart' hide Route;
```

---

## Legal Requirements

### Feature restrictions by region

Some features may be illegal in certain countries:

- **Safety features:** Disable the `safety` overlay and related `socialReports` entries where police/speed camera reporting is prohibited

- **Social overlays:** Restrict `SocialOverlay` features based on local regulations

### Data attribution

**OpenStreetMap:** The SDK uses OpenStreetMap data. Provide proper attribution per the [OpenStreetMap license](https://osmfoundation.org/wiki/Licence).

**Wikipedia:** Display appropriate attribution when using Wikipedia content, following [Reusers' rights and obligations](https://en.wikipedia.org/wiki/Wikipedia:Copyrights#Reusers'_rights_and_obligations).
