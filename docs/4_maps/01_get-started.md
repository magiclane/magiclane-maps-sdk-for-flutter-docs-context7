---
description: Documentation for Get Started
title: Get Started
---

# Get started with maps

The Maps SDK for Flutter delivers powerful mapping capabilities, enabling developers to effortlessly integrate dynamic map views into their applications. Core features include embedding and customizing map views, controlling displayed locations, and fine-tuning map properties. At the center of the mapping API is the `GemMap`, a subclass of `StatefulWidget`, offering a wide range of configurable options.

## Display a map

The following code demonstrates how to show a map view. This is the ``main.dart`` file.
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
      body: const GemMap(
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

The `GemMap` widget includes a callback, ``onMapCreated``, which is triggered once the map has finished initializing. It also provides a ``GemMapController`` to enable additional map functionalities. 
```dart
const GemMap(
  appAuthorization: projectApiToken,
  //highlight-start
  onMapCreated: _onMapCreated,
  //highlight-end
),
```

Multiple ``GemMap`` widgets can be instantiated within a single application, allowing for the display of different data on each map. Each ``GemMap`` is independently controlled via its respective ``GemMapController``.

Note that certain settings, such as language, overlay visibility, and position tracking, are shared across all GemMap instances within the application.

## Relevant examples demonstrating map related features

- [Map Compass](/examples/maps-3dscene-examples/map-compass)

- [Map Perspective](/examples/maps-3dscene-examples/map-perspective)

- [Center Coordinates](/examples/maps-3dscene-examples/center-coordinates)

- [Map Gestures](/examples/maps-3dscene-examples/map-gestures)

- [Landmarks Selection](/examples/maps-3dscene-examples/map-selection)

- [Display Cursor Street Name](/examples/places-search/display-cursor-street)

- [Map Styles](/examples/maps-3dscene-examples/map-styles)

- [Assets Map Style](/examples/maps-3dscene-examples/assets-map-styles)
