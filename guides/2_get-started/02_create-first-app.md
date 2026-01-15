---
description: Documentation for Create First App
title: Create First App
---

# Create your first application

Follow this tutorial to build a Flutter app with an interactive map.

- Flutter installed ([installation guide](https://docs.flutter.dev/get-started/install))

- [Free API key from MagicLane](https://developer.magiclane.com/docs/guides/get-started)

---

## Step 1: Create a new project
```bash
flutter create my_first_map_app
cd my_first_map_app
```

<details>
    <summary>Using an existing project?</summary>
    
Skip to Step 2 and add the SDK to your existing app.
</details>

## Step 2: Install the SDK

Add the package to `pubspec.yaml`:
```yaml
dependencies:
  flutter:
    sdk: flutter
  magiclane_maps_flutter:  # Add this line
```

Install it:
```bash
flutter pub get
```

Complete the [Android/iOS setup](./integrate-sdk#step-2-configure-your-platform) before continuing (adds Maven repository, sets iOS version).

## Step 3: Write the code

Open `lib/main.dart` and replace everything with this:
```dart
import 'package:flutter/material.dart' hide Route;
import 'package:magiclane_maps_flutter/magiclane_maps_flutter.dart';

const projectApiToken = String.fromEnvironment('GEM_TOKEN');

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Hello Map',
      home: MyHomePage(),
    );
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key});

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  @override
  void dispose() {
    GemKit.release(); // Clean up SDK resources
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        backgroundColor: Colors.deepPurple[900],
        title: const Text('Hello Map', style: TextStyle(color: Colors.white)),
      ),
      body: GemMap(
        appAuthorization: projectApiToken,
        onMapCreated: _onMapCreated,
      ),
    );
  }

  void _onMapCreated(GemMapController mapController) {
    // Map is initialized and ready to use
  }
}
```

<details>
    <summary>Understanding the code</summary>

- **`GemMap`** - Widget that displays the interactive map

- **`appAuthorization`** - Your API key for SDK authentication

- **`onMapCreated`** - Called when the map finishes loading

- **`GemKit.release()`** - Frees memory when the app closes

</details>

---

## Step 4: Run your app

Start the app with your API key:
```bash
flutter run --dart-define=GEM_TOKEN="your_actual_token_here"
```

<details>
    <summary>Quick test: Hardcode the token</summary>

For quick testing only, change line 4 to:
```dart
const projectApiToken = "your_actual_token_here";
```

**⚠️ Never commit hardcoded tokens to version control!**
</details>

---

🎉 **Success!** Your map app is running.

---

## What's next?

Explore what you can do with the map:

- [Add markers](../../examples/maps-3dscene/add-markers) - Pin locations on the map

- [Draw shapes](../../examples/maps-3dscene/draw-shapes) - Add polygons, lines, and circles

- [Change map styles](../../examples/maps-3dscene/map-styles) - Switch between light, dark, and custom themes

- [Search for places](../../examples/places-search) - Find addresses and points of interest

- [Add navigation](../../examples/routing-navigation) - Calculate routes and turn-by-turn directions

---

## Troubleshooting

<details>
    <summary>App crashes immediately</summary>

**Most common causes:**

1. **Missing platform setup** - Did you complete the [Android/iOS configuration](./integrate-sdk#step-2-configure-your-platform)?
2. **Invalid API key** - Check your token is correct and active
3. **Flutter environment issues** - Run `flutter doctor` and fix any issues

</details>

<details>
    <summary>GemKitUninitializedException error</summary>

You're calling SDK methods before the map initializes.

**Solution:** Only call SDK methods inside `onMapCreated` or after the map loads.
```dart
void _onMapCreated(GemMapController mapController) {
  // ✓ Safe to call SDK methods here
}
```

</details>

<details>
    <summary>How to check if my API key is valid</summary>

Add this code to verify your token:
```dart
SdkSettings.verifyAppAuthorization(projectApiToken, (status) {
  if (status == GemError.success) {
    print('✓ API key is valid');
  } else {
    print('✗ API key error: $status');
  }
});
```

**Possible status codes:**

- `success` - Token is valid ✓

- `invalidInput` - Wrong format ✗

- `expired` - Token expired ✗

- `accessDenied` - Token blocked ✗

</details>

<details>
    <summary>Map shows a watermark</summary>

This means your API key is missing or invalid.

**Without a valid API key:**

- ❌ Map downloads won't work

- ❌ Map updates disabled

- ⚠️ Watermark appears

- ⚠️ Limited features

**Fix:** Ensure you're passing a valid API key via `--dart-define=GEM_TOKEN="your_token"`

</details>

<details>
    <summary>Advanced: Manual SDK initialization</summary>

The SDK auto-initializes when you create a `GemMap` widget.

If you need to use SDK features **before** showing a map:
```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await GemKit.initialize(appAuthorization: projectApiToken);
  
  runApp(const MyApp());
}

// Don't forget to clean up!
@override
void dispose() {
  GemKit.release();
  super.dispose();
}
```

**When to use this:**

- Background location tracking before map display

- Preloading map data

- SDK services without showing a map

</details>
