---
description: Documentation for Create First App
title: Create First App
---

# Create your first application

Prerequisites:

- Have an API key - if you don't, take a look at [Create API key](./create-api-key).

- Integrate the SDK in the current application - if you didn't yet, see [Integrate the SDK](./integrate-sdk).

## Create a simple Flutter app (without map)

Sometimes we don't need to display a map, but still want to access functionalities like search, routing etc.

In such cases we first need to initialize the engine before using SDK functionalities. Also, at the end we need to free the resources (release the SDK).

We can do this using the following code:
```dart
import 'package:gem_kit/core.dart';

const projectApiToken = String.fromEnvironment('GEM_TOKEN');

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await GemKit.initialize(appAuthorization: projectApiToken);

  // your logic

  GemKit.release();
}
```

You can also send the token directly without getting it from the environment:
```dart
import 'package:gem_kit/core.dart';

const String projectApiToken = "YOUR_API_TOKEN";

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await GemKit.initialize(appAuthorization: projectApiToken);

  // your logic

  GemKit.release();
}
```

Any call to a SDK method before the engine is initialized will throw an `GemKitUninitializedException`.

App authorization can also be done with `SdkSettings.appAuthorization` setter.

## Create a Flutter app with a map

In order to create an application with a map you can write the following code:
```dart
import 'package:flutter/material.dart';

import 'package:gem_kit/core.dart';
import 'package:gem_kit/map.dart';

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
    GemKit.release();
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
    // Code executed when map initialized
  }
}
```

Run your project in either mode, substituting `your_token` with your actual token:
```bash
# Debug mode
flutter run --debug --dart-define=GEM_TOKEN="your_token"
# Release mode
flutter run --release --dart-define=GEM_TOKEN="your_token"
```

Any call to a SDK method before the engine is initialized will throw an `GemKitUninitializedException`.

When initializing a `GemMap`, there’s no need to explicitly call `GemKit.initialize`—this is handled automatically during map creation. Additionally, if your app includes multiple `GemMap` instances, you only need to provide an authorization token for the first one. Authorization is completed during the creation of the initial `GemMap`, so passing a token to subsequent instances will result in a `GemKitUninitializedException`.

Some explanations:

- The `MyHomePage` widget contains the scaffold that houses the map.

- The `dispose` method ensures that resources are released when the widget is destroyed.

- The `GemMap` widget is used to display the interactive map in the body of the scaffold.

If the API key is not configured, some features will be restricted, and a watermark will appear on the map. Functionality such as map downloads, updates, and other features may be unavailable or may not function as expected. Please ensure the API token is set correctly. Ensure the API key is stored securely and protected against unauthorized access or exposure.

The method ``verifyAppAuthorization`` from the ``SdkSettings`` class might be used to check if the token is set:
```dart
SdkSettings.verifyAppAuthorization(token, (status) {
  switch (status) {
    case GemError.success:
      print('The token is set and is valid.');
      break;
    case GemError.invalidInput:
      print('The token is invalid.');
      break;
    case GemError.expired:
      print('The token is expired.');
      break;
    case GemError.accessDenied:
      print('The token is blacklisted.');
      break;
    default:
      print('Other error regarding token validation : $status.');
      break;
  }
})
```


