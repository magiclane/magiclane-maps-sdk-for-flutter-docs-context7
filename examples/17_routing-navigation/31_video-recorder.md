---
description: Documentation for Video Recorder
title: Video Recorder
---

# Video Recorder

This example demonstrates how to build a Flutter app using the Maps SDK to record video (in chunks) with audio and display the user's track on the map.

## How it works

The example app highlights the following features:

- Initializing a map.

- Requesting and handling camera, microphone, and location permissions.

- Starting and stopping video recording (with configurable chunk duration).

- Pausing and resuming audio recording during the session.

- Displaying the recorded path on the map once recording stops.

### UI and Map Integration

The following code builds the UI with a `GemMap` widget and an app bar that includes buttons for starting/stopping video recording, controlling audio recording, and following the user's position.
```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Video Recorder',
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
  late GemMapController _mapController;
  late Recorder _recorder;

  PermissionStatus _locationPermissionStatus = PermissionStatus.denied;
  bool _hasLiveDataSource = false;
  bool _isRecording = false;
  bool _isAudioRecording = false;

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
        title: const Text('Video Recorder', style: TextStyle(color: Colors.white)),
        actions: [
          if (_hasLiveDataSource && !_isRecording)
            IconButton(
              onPressed: _onRecordButtonPressed,
              icon: const Icon(Icons.radio_button_on, color: Colors.white),
            ),
          if (_isRecording)
            IconButton(
              onPressed: _onStopRecordingButtonPressed,
              icon: const Icon(Icons.stop_circle, color: Colors.white),
            ),
          if (_isRecording)
            IconButton(
              onPressed: _startAudioRecording,
              icon: Icon(Icons.mic, color: _isAudioRecording ? Colors.green : Colors.white),
            ),
          if (_isRecording)
            IconButton(
              onPressed: _stopAudioRecording,
              icon: Icon(Icons.mic_off, color: _isAudioRecording ? Colors.white : Colors.grey),
            ),
          IconButton(
            onPressed: _onFollowPositionButtonPressed,
            icon: const Icon(Icons.location_searching_sharp, color: Colors.white),
          ),
        ],
      ),
      body: Stack(
        children: [
          GemMap(
            key: const ValueKey("GemMap"),
            onMapCreated: (controller) => _onMapCreated(controller),
            appAuthorization: projectApiToken,
          ),
        ],
      ),
    );
  }

  void _onMapCreated(GemMapController controller) {
    _mapController = controller;
  }
}
```

### Requesting Permissions

The following code requests location permission (and storage permission on web) and then camera & microphone permissions before starting a recording.
```dart
Future<void> _onFollowPositionButtonPressed() async {
  if (kIsWeb) {
    final locationPermssionWeb = await PositionService.requestLocationPermission();
    _locationPermissionStatus = locationPermssionWeb == true
        ? PermissionStatus.granted
        : PermissionStatus.denied;
  } else {
    _locationPermissionStatus = await Permission.locationWhenInUse.request();
  }

  if (_locationPermissionStatus == PermissionStatus.granted) {
    if (!_hasLiveDataSource) {
      PositionService.setLiveDataSource();
      _hasLiveDataSource = true;
    }
    final animation = GemAnimation(type: AnimationType.linear);
    _mapController.startFollowingPosition(animation: animation);
    setState(() {});
  }
}

Future<bool> requestCameraAndMicPermissions() async {
  final cameraStatus = await Permission.camera.request();
  final micStatus = await Permission.microphone.request();
  return cameraStatus.isGranted && micStatus.isGranted;
}
```

### Starting and Stopping Recording
```dart
Future<void> _onRecordButtonPressed() async {
  final hasCamMicPermission = await requestCameraAndMicPermissions();
  if (!hasCamMicPermission) {
    if (mounted) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(
          content: Text('Camera or microphone permission not granted.'),
          duration: Duration(seconds: 3),
        ),
      );
    }
    return;
  }

  final logsDir = await getDirectoryPath("Tracks");
  final recorder = Recorder.create(
    RecorderConfiguration(
      dataSource: DataSource.createLiveDataSource()!,
      logsDir: logsDir,
      recordedTypes: [
        DataType.position,
        DataType.camera,
      ],
      enableAudio: true,
      minDurationSeconds: 5,
      videoQuality: Resolution.hd720p,
      chunkDurationSeconds: 180,
    ),
  );

  setState(() {
    _isRecording = true;
    _recorder = recorder;
  });

  final error = await _recorder.startRecording();
  if (error != GemError.success) {
    if (mounted) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: Text('Recording failed: $error'),
          duration: const Duration(seconds: 5),
        ),
      );
    }
    setState(() => _isRecording = false);
    return;
  }

  _mapController.preferences.paths.clear();
  _mapController.deactivateAllHighlights();
}

Future<void> _onStopRecordingButtonPressed() async {
  final endErr = await _recorder.stopRecording();
  if (endErr == GemError.success) {
    await _presentRecordedRoute();
  } else if (mounted) {
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        content: Text('Recording failed: $endErr'),
        duration: const Duration(seconds: 5),
      ),
    );
  }
  setState(() => _isRecording = false);
}
```

The resulting video recordings are saved as `.mp4` files in the `Data/Tracks` directory specified in the recorder configuration.

### Starting and Stopping Audio Recording
```dart
void _startAudioRecording() {
  _recorder.startAudioRecording();
  setState(() => _isAudioRecording = true);
}

void _stopAudioRecording() {
  _recorder.stopAudioRecording();
  setState(() => _isAudioRecording = false);
}
```

### Presenting the Recorded Track on the Map

This code loads the last recorded track from device memory, retrieves the coordinates, builds a `Path` entity, and adds it to the `MapViewPathCollection`.
```dart
  Future<void> _presentRecordedRoute() async {
    // The recorded tracks are stored in /Data/Tracks directory
    final logsDir = await getDirectoryPath("Tracks");

    // It loads all .gm and .mp4 files at logsDir
    final bookmarks = RecorderBookmarks.create(logsDir);

    // Get all recordings path
    final logList = bookmarks?.logsList;

    // Get the LogMetadata to obtain details about recorded session
    LogMetadata? meta = bookmarks!.getLogMetadata(logList!.last);
    if (meta == null) {
      // Handle the case where metadata is not found
      return;
    }
    final recorderCoordinates = meta.preciseRoute;
    final duration = convertDuration(meta.durationMillis);

    // Create a path entity from coordinates
    final path = Path.fromCoordinates(recorderCoordinates);

    Landmark beginLandmark = Landmark.withCoordinates(recorderCoordinates.first);
    Landmark endLandmark = Landmark.withCoordinates(recorderCoordinates.last);

    beginLandmark.setImageFromIcon(GemIcon.waypointStart);
    endLandmark.setImageFromIcon(GemIcon.waypointFinish);

    HighlightRenderSettings renderSettings = HighlightRenderSettings(
      options: {HighlightOptions.showLandmark},
    );

    _mapController.activateHighlight([beginLandmark, endLandmark], renderSettings: renderSettings, highlightId: 1);

    // Show the path immediately after stopping recording
    _mapController.preferences.paths.add(path);

    // Center on recorder path
    _mapController.centerOnAreaRect(
      path.area,
      viewRc: Rectangle<int>(
        _mapController.viewport.width ~/ 3,
        _mapController.viewport.height ~/ 3,
        _mapController.viewport.width ~/ 3,
        _mapController.viewport.height ~/ 3,
      ),
    );

    if (mounted) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: Text('Duration: $duration'),
          duration: Duration(seconds: 5),
        ),
      );
    }
  }
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

// Utility function to convert the seconds duration into a suitable format
String convertDuration(int milliseconds) {
  int totalSeconds = (milliseconds / 1000).floor();
  int hours = totalSeconds ~/ 3600;
  int minutes = (totalSeconds % 3600) ~/ 60;
  int seconds = totalSeconds % 60;

  String hoursText = (hours > 0) ? '$hours h ' : '';
  String minutesText = (minutes > 0) ? '$minutes min ' : '';
  String secondsText = '$seconds sec';

  return hoursText + minutesText + secondsText;
}

```


