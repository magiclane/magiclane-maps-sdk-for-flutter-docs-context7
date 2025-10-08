---
description: Documentation for Overlapped Maps
title: Overlapped Maps
---

# Overlapped Maps

This example demonstrates how to create a Flutter application that utilizes the Maps SDK for Flutter to display overlapped maps. The application initializes the GemKit SDK and renders two maps on top of each other, showcasing the ability to layer map views.

## How it works

- Main App Setup : Sets up the app’s home screen with two maps overlapped.

### UI and Map Integration

The main application consists of a simple user interface that displays two maps stacked on top of each other. The user can see both maps simultaneously, allowing for comparison or overlaying of different data. This code sets up the main application UI, including an app bar and a body that contains a stack of maps.
```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Overlapped Maps',
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
        title: const Text('Overlapped Maps', style: TextStyle(color: Colors.white)),
      ),
      // Stack maps
      body: Stack(children: [
          const GemMap(
            key: ValueKey("GemMap"),
            appAuthorization: projectApiToken,
          ),
          SizedBox(
          height: MediaQuery.of(context).size.height * 0.4,
          width: MediaQuery.of(context).size.width * 0.4,
          child: const GemMap(appAuthorization: projectApiToken),
        ),
      ]),
    );
  }
}
```


