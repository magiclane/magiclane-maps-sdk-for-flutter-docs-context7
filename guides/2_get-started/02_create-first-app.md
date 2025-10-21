---
description: Documentation for Create First App
title: Create First App
---

# Create your first application

The guide will walk you through the steps to create a simple Flutter application that displays a map using the MagicLane Maps SDK.

For this guide you will need to have an API key - follow our [step-by-step guide](https://developer.magiclane.com/docs/guides/get-started) to sign up for a free account, create a project and generate your API key.

## Create a Flutter project

Make sure you have Flutter installed on your machine and `flutter doctor` shows no issues.
It is recommended to use the latest stable version of Flutter.

To create a new Flutter project, run the following command in your terminal, replacing `my_first_map_app` with your desired project name:
 
```bash
flutter create my_first_map_app
```

If you already have a Flutter project, you can skip this step.

## Add the MagicLane Maps Flutter dependency

To add the MagicLane Maps SDK to your Flutter project, open the `pubspec.yaml` file and add the following dependency:
```yaml
dependencies:
  magiclane_maps_flutter: 
```

Additional platform-specific setup is required. Refer to the [Installation guide](./integrate-sdk#native-configuration) for detailed instructions.

## Write the application code

To create an application with a map, use the following code:
```dart
import 'package:flutter/material.dart' hide Route; // To avoid conflict with the Route class from magiclane_maps_flutter
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
    // Code executed when the map is initialized
  }
}
```

Run your project in either mode, replacing `your_token` with your actual token:
```bash
# Debug mode
flutter run --debug --dart-define=GEM_TOKEN="your_token"
# Release mode
flutter run --release --dart-define=GEM_TOKEN="your_token"
```

Or you can replace `String.fromEnvironment('GEM_TOKEN');` with your token directly in the code.

Some explanations:

- The `MyHomePage` widget contains the scaffold that houses the map.

- The `dispose` method ensures that resources are released when the widget is destroyed.

- The `GemMap` widget is used to display the interactive map in the body of the scaffold.

Any call to a SDK method before the engine is initialized will throw an `GemKitUninitializedException`.

The SDK is automatically initialized when creating a `GemMap` widget. However, if you need to initialize it manually (for example, if you want to use the SDK without displaying a map or some operations are needed before a map is shown), you can do so by calling `GemKit.initialize`:
```dart
WidgetsFlutterBinding.ensureInitialized();
await GemKit.initialize(appAuthorization: projectApiToken);
```

It is also recommended to call `await GemKit.release` when the SDK is no longer needed to free up resources.

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


