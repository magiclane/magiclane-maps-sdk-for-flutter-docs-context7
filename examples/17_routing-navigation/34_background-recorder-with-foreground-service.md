---
description: Documentation for Background Recorder With Foreground Service
title: Background Recorder With Foreground Service
---

# Background Recorder with Foreground Service (Android only)

This example demonstrates how to build a Flutter app using the Maps SDK to record a user’s track while running in the **background**.  
The app integrates with the **Android foreground service** and **notifications API** to keep recording active even when the app is not visible.

## How it works

The example app highlights the following features:

- Requesting **location** and **notification** permissions.

- Initializing a **foreground service** to enable background location updates.

- Recording a track and saving it to device storage.

- Displaying the recorded track and duration on the map.

---

### UI and Map Integration

The following code builds the UI with a `GemMap` widget and app bar controls for recording and following the user’s position.
```dart
class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key});

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  late GemMapController _mapController;
  late Recorder _recorder;

  PermissionStatus _locationPermissionStatus = PermissionStatus.denied;
  PermissionStatus _notificationPermissionStatus = PermissionStatus.denied;

  bool _hasLiveDataSource = false;
  bool _isInitialized = false;
  bool _isRecording = false;
  bool _isRecorder = false;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        backgroundColor: Colors.deepPurple[900],
        title: const Text('Background Location', style: TextStyle(color: Colors.white)),
        actions: [
          if (_hasLiveDataSource && _isRecording == false)
            IconButton(
              onPressed: _startRecording,
              icon: Icon(Icons.radio_button_on, color: Colors.white),
            ),
          if (_isRecording)
            IconButton(
              onPressed: _onStopRecordingButtonPressed,
              icon: Icon(Icons.stop_circle, color: Colors.white),
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
            key: ValueKey("GemMap"),
            onMapCreated: (controller) => _onMapCreated(controller),
            appAuthorization: projectApiToken,
          ),
        ],
      ),
    );
  }
```

### Foreground Service Setup

The AndroidForegroundService class wraps the flutter_background_service and flutter_local_notifications plugins.
It initializes the notification channel, requests permissions, and manages the lifecycle of the foreground service.
```dart
@pragma('vm:entry-point')
class AndroidForegroundService {
  static final service = FlutterBackgroundService();
  static final notificationsPlugin = FlutterLocalNotificationsPlugin();
  static final notificationId = 888;

  static Future<void> initialize(bool isForegroundMode) async {
    const initSettings = InitializationSettings(
      android: AndroidInitializationSettings('@mipmap/ic_launcher'),
    );
    await notificationsPlugin.initialize(initSettings);

    channel = AndroidNotificationChannel(
      notificationId.toString(),
      'MY FOREGROUND SERVICE',
      description: 'Used for background location.',
      importance: Importance.low,
    );

    // Request notification permission
    hasGrantedNotificationsPermission =
        await notificationsPlugin.resolvePlatformSpecificImplementation<
                AndroidFlutterLocalNotificationsPlugin>()
            ?.requestNotificationsPermission() ??
        false;

    if (!hasGrantedNotificationsPermission) return;

    await notificationsPlugin
        .resolvePlatformSpecificImplementation<
            AndroidFlutterLocalNotificationsPlugin>()
        ?.createNotificationChannel(channel);

    await service.configure(
      androidConfiguration: AndroidConfiguration(
        onStart: onStart,
        autoStart: false,
        isForegroundMode: isForegroundMode,
        notificationChannelId: notificationId.toString(),
        foregroundServiceNotificationId: notificationId,
        initialNotificationTitle: 'Background location',
        initialNotificationContent: 'Background location is active',
        foregroundServiceTypes: [AndroidForegroundType.location],
      ),
      iosConfiguration: IosConfiguration(),
    );
  }

  static Future<bool> start() async => await service.startService();
  static Future<void> stop() async => service.invoke("stopService");
}

```

### Requesting Permissions

The app requests both notification and background location permissions before enabling recording and the foreground service.
```dart
Future<void> _onFollowPositionButtonPressed() async {
  // Request notification permission
  if (_notificationPermissionStatus != PermissionStatus.granted) {
    _notificationPermissionStatus = await Permission.notification.request();
  }

  // Request location permissions
  final whenInUseStatus = await Permission.locationWhenInUse.request();
  if (whenInUseStatus == PermissionStatus.granted) {
    _locationPermissionStatus = await Permission.locationAlways.request();
  } else {
    _locationPermissionStatus = whenInUseStatus;
  }

  if (_locationPermissionStatus == PermissionStatus.granted) {
    if (!_hasLiveDataSource) {
      PositionService.instance.setLiveDataSource();
      _hasLiveDataSource = true;
    }

    final animation = GemAnimation(type: AnimationType.linear);
    _mapController.startFollowingPosition(animation: animation);
    setState(() {});
  }
}
```

### Starting and Stopping Recording

Recording only starts after the foreground service is initialized.
When stopped, the service is shut down and the recorded track is presented.
```dart
Future<void> _startRecording() async {
  if (!_isInitialized) {
    await _initializeForegroundService();
  }

  AndroidForegroundService.start();

  if (!_isRecorder) {
    await _createRecorder();
  }

  await _recorder.startRecording();

  // Reset map state
  _mapController.preferences.paths.clear();
  _mapController.deactivateAllHighlights();

  setState(() => _isRecording = true);
}

Future<void> _onStopRecordingButtonPressed() async {
  final endErr = await _recorder.stopRecording();

  if (endErr == GemError.success) {
    await _presentRecordedRoute();
  } else {
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text('Recording failed: $endErr')),
    );
  }

  AndroidForegroundService.stop();
  setState(() => _isRecording = false);
}
```

### Presenting the Recorded Track

When recording is complete, the last session is loaded from disk and displayed on the map.
```dart
Future<void> _presentRecordedRoute() async {
  final logsDir = await getDirectoryPath("Tracks");
  final bookmarks = RecorderBookmarks.create(logsDir);

  final logList = bookmarks?.getLogsList();
  LogMetadata? meta = bookmarks!.getLogMetadata(logList!.last);

  if (meta == null) return;

  final recorderCoordinates = meta.preciseRoute;
  final duration = convertDuration(meta.durationMillis);

  final path = Path.fromCoordinates(recorderCoordinates);
  final beginLandmark = Landmark.withCoordinates(recorderCoordinates.first);
  final endLandmark = Landmark.withCoordinates(recorderCoordinates.last);

  beginLandmark.setImageFromIcon(GemIcon.waypointStart);
  endLandmark.setImageFromIcon(GemIcon.waypointFinish);

  _mapController.activateHighlight([beginLandmark, endLandmark],
      renderSettings: HighlightRenderSettings(
        options: {HighlightOptions.showLandmark},
      ),
      highlightId: 1);

  _mapController.preferences.paths.add(path);
  _mapController.centerOnAreaRect(path.area, viewRc: RectType(
    x: _mapController.viewport.width ~/ 3,
    y: _mapController.viewport.height ~/ 3,
    width: _mapController.viewport.width ~/ 3,
    height: _mapController.viewport.height ~/ 3,
  ));

  ScaffoldMessenger.of(context).showSnackBar(
    SnackBar(content: Text('Duration: $duration')),
  );
}
```

### Utility Functions
```dart
Future<String> getDirectoryPath(String dirName) async {
  final docDirectory = Platform.isAndroid
      ? await path_provider.getExternalStorageDirectory()
      : await path_provider.getApplicationDocumentsDirectory();

  final absPath = docDirectory!.path;
  return path.joinAll([absPath, "Data", dirName]);
}

String convertDuration(int milliseconds) {
  int totalSeconds = milliseconds ~/ 1000;
  int hours = totalSeconds ~/ 3600;
  int minutes = (totalSeconds % 3600) ~/ 60;
  int seconds = totalSeconds % 60;

  String hoursText = (hours > 0) ? '$hours h ' : '';
  String minutesText = (minutes > 0) ? '$minutes min ' : '';
  String secondsText = (hours == 0 && minutes == 0) ? '$seconds sec' : '';

  return (hoursText + minutesText + secondsText).trim();
}
```


