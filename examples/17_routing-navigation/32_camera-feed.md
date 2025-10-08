---
description: Documentation for Camera Feed
title: Camera Feed
---

# Camera Feed

This example demonstrates how to build a Flutter app using the Maps SDK to display a live camera feed (while recording) and overlay it on the map.

## How it works

The example app highlights the following features:

- Initializing the GemKit SDK.

- Requesting and handling camera, microphone, and location permissions.

- Starting and stopping video recording with the device’s camera.

- Streaming the live camera frames to a `GemCameraPlayer` widget.

- Overlaying the camera preview on top of the map.

- Properly disposing of resources when recording stops or the app is closed.

### UI and Map Integration

The following code builds the UI with a `GemMap` widget and an app bar that includes buttons for starting/stopping recording and streaming the camera feed.
```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});
  
  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Camera Feed',
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
  Recorder? _recorder;
  DataSource? _ds;
  GemCameraPlayerController? _cameraPlayerController;
  
  @override
  void initState() {
    super.initState();
  }
  
  @override
  void dispose() {
    GemKit.release();
    super.dispose();
  }
  
  void _watchPlayerStatus() {
    _cameraPlayerController?.addListener(() {
      if (_cameraPlayerController!.status == GemCameraPlayerStatus.playing) {
        setState(() {});
      }
    });
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        backgroundColor: Colors.deepPurple[900],
        title: const Text('Camera Feed', style: TextStyle(color: Colors.white)),
        actions: [
          IconButton(
            icon: const Icon(Icons.play_arrow, color: Colors.white),
            onPressed: _startRecordingAndFeed,
          ),
          IconButton(
            icon: const Icon(Icons.stop, color: Colors.white),
            onPressed: _stopRecordingAndFeed,
          ),
        ],
      ),
      body: Stack(
        children: [
          GemMap(
            key: const ValueKey("GemMap"),
            onMapCreated: _onMapCreated,
            appAuthorization: projectApiToken,
          ),
          _buildCameraOverlay(),
        ],
      ),
    );
  }
}
```

### Requesting Location Permission

Before streaming or recording, the app requests camera, microphone, and location permissions:
```dart

void _onMapCreated(GemMapController controller) async {
  await [
    Permission.camera,
    Permission.microphone,
    Permission.locationWhenInUse,
  ].request();
}

```

### Starting Recording and Camera Feed

Tapping the Play button initializes the live data source, starts recording, and sets up the `GemCameraPlayerController` to stream frames:
```dart
   onPressed: () async {
        if (_cameraPlayerController == null) {
            _ds = DataSource.createLiveDataSource()!;

            _ds!.start();

            _recorder = Recorder.create(RecorderConfiguration(
            logsDir: await getDirectoryPath('Tracks'),
            dataSource: _ds!,
            videoQuality: Resolution.hd720p,
            recordedTypes: [DataType.position, DataType.camera],
            transportMode: RecordingTransportMode.car,
            ));

            await _recorder!.startRecording();

            _cameraPlayerController = GemCameraPlayerController(dataSource: _ds!);

            _watchPlayerStatus();

            setState(() {});
        }
    },
```

### Stopping Recording and Disposing Resources

Tapping the Stop button stops the recorder, disposes the player, and stops the data source:
```dart
    onPressed: () async {
        await _recorder!.stopRecording();

        _cameraPlayerController?.dispose();
        _ds?.stop();
        _cameraPlayerController = null;
        _ds = null;
        setState(() {});
    },
```

### Stopping Recording and Disposing Resources

A small, resizable preview of the camera feed is overlaid in the top-left corner using `GemCameraPlayer`:
```dart
    Positioned(
        top: 10,
        left: 10,
        child: SafeArea(
        top: true,
        left: true,
        child: Builder(
            builder: (context) {
            final controller = _cameraPlayerController;

            if (controller == null ||
                controller.isDisposed ||
                controller.status != GemCameraPlayerStatus.playing ||
                controller.size == null) {
                return const SizedBox(
                width: 150,
                height: 150,
                child: Center(child: CircularProgressIndicator()),
                );
            }

            return SizedBox(
                width: 200,
                child: AspectRatio(
                aspectRatio: controller.size!.$1.toDouble() / controller.size!.$2.toDouble(),
                child: GemCameraPlayer(
                    controller: controller,
                    fit: BoxFit.cover,
                ),
                ),
            );
            },
        ),
        ),
    )
```

### Utility Functions

The `getDirectoryPath` function retrieves the root directory path for the app and returns the desired directory path inside the "Data" folder.
```dart
import 'package:path_provider/path_provider.dart' as path_provider;
import 'package:path/path.dart' as path;

import 'dart:io';

Future<String> getDirectoryPath(String dirName) async {
  final docDirectory =
      Platform.isAndroid
          ? await path_provider.getExternalStorageDirectory()
          : await path_provider.getApplicationDocumentsDirectory();

  String absPath = docDirectory!.path;

  final expectedPath = path.joinAll([absPath, "Data", dirName]);
  return expectedPath;
}
```


