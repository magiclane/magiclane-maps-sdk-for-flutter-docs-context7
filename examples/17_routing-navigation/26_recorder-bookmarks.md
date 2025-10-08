---
description: Documentation for Recorder Bookmarks
title: Recorder Bookmarks
---

# Recorder Bookmarks

This guide will teach you how to display recorded logs data by using `RecorderBookmarks` class.

## Saving Assets

Before running the app, ensure that you save the necessary file or files (`.gm` file) into the assets directory.

Update your `pubspec.yaml` file to include these assets:
```yaml
flutter:
  assets:
    - assets/
```

## How it works

The example app demonstrates the following key features:

- Import `Recorder` logs from assets folder to app documents directory.

- Display logs metadata via `LogMetadata` class.

- Delete logs.

### UI and Map Integration

The following code builds the UI with a `GemMap` widget and an app bar containing a import log files button. Once the logs are imported, a `RecorderBookmarks` object is created and a button will appear on bottom of the screen. The recorder logs button will navigate to the `RecorderBookmarksPage`.
```dart
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
      title: 'Recorder Bookmarks',
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
  RecorderBookmarks? _recorderBookmarks;

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
        title: const Text('Recorder Bookmarks', style: TextStyle(color: Colors.white)),
        actions: [
          if (_recorderBookmarks == null)
            IconButton(
              onPressed: _onImportButtonPressed,
              icon: Icon(Icons.upload, color: Colors.white),
            ),
        ],
      ),
      body: Stack(
        children: [
          GemMap(
            key: ValueKey("GemMap"),
            appAuthorization: projectApiToken,
          ),
          if (_recorderBookmarks != null)
            Positioned(
              bottom: 15,
              left: 0,
              right: 0,
              child: ElevatedButton(
                  onPressed: () {
                    Navigator.of(context).push(
                      MaterialPageRoute<void>(
                        builder: (context) {
                          return RecorderBookmarksPage(recorderBookmarks: _recorderBookmarks!);
                        },
                      ),
                    );
                  },
                  child: Text("Recorder Logs")),
            ),
        ],
      ),
    );
  }

  Future<void> _onImportButtonPressed() async {
    // Upload log files from assets folder to phone's memory into Tracks directory
    copyLogToAppDocsDir("2025-04-29_12-42-26_700.gm");
    copyLogToAppDocsDir("2025-04-29_12-59-52_568.gm");

    // Get Tracks directory path
    final logsDirectory = await getDirectoryPath("Tracks");

    // Create a RecorderBookmarks instance based on Tracks directory location
    final recorderBookmarks = RecorderBookmarks.create(logsDirectory);

    if (recorderBookmarks == null) {
      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(
            content: Text('Error while creating RecorderBookmarks'),
            duration: Duration(seconds: 3),
          ),
        );
      }
      return;
    }

    setState(() {
      _recorderBookmarks = recorderBookmarks;
    });

    // ignore: use_build_context_synchronously
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        content: Text('Successfully imported logs.'),
        duration: Duration(seconds: 3),
      ),
    );
  }
}
```

### Recorder Bookmarks Page
```dart
class RecorderBookmarksPage extends StatefulWidget {
  final RecorderBookmarks recorderBookmarks;

  const RecorderBookmarksPage({super.key, required this.recorderBookmarks});

  @override
  State<RecorderBookmarksPage> createState() => _RecorderBookmarksPageState();
}

class _RecorderBookmarksPageState extends State<RecorderBookmarksPage> {
  late List<String> _logs;

  @override
  void initState() {
    super.initState();
    _logs = widget.recorderBookmarks.getLogsList();
  }

  void _deleteLogAt(int index) {
    final removed = _logs.removeAt(index);
    widget.recorderBookmarks.deleteLog(removed);
    setState(() {});
  }

  @override
  Widget build(BuildContext context) {
    final theme = Theme.of(context);
    return Scaffold(
      appBar: AppBar(
        title: const Text(
          'Recordings',
          style: TextStyle(color: Colors.white),
        ),
        backgroundColor: Colors.deepPurple[900],
        elevation: 0,
        iconTheme: IconThemeData(color: theme.colorScheme.onPrimary),
      ),
      body: ListView.separated(
        itemCount: _logs.length,
        separatorBuilder: (_, __) => const Divider(height: 1),
        itemBuilder: (context, i) {
          final metadata = widget.recorderBookmarks.getLogMetadata(_logs[i]);
          if (metadata == null) {
            return const SizedBox.shrink();
          }
          return LogItem(
            logMetadata: metadata,
            onDelete: () => _deleteLogAt(i),
          );
        },
      ),
    );
  }
}

class LogItem extends StatelessWidget {
  final VoidCallback onDelete;
  const LogItem({
    super.key,
    required this.logMetadata,
    required this.onDelete,
  });

  final LogMetadata logMetadata;

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 12.0, horizontal: 16.0),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
          Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Text(
                'Start: ${DateTime.fromMillisecondsSinceEpoch(
                  logMetadata.startTimestampInMillis,
                ).toLocal()}',
              ),
              const SizedBox(height: 4),
              Text(
                'End: ${DateTime.fromMillisecondsSinceEpoch(
                  logMetadata.endTimestampInMillis,
                ).toLocal()}',
              ),
              const SizedBox(height: 4),
              Text(
                'Duration: ${Duration(milliseconds: logMetadata.durationMillis)}',
              ),
              const SizedBox(height: 4),
              Text(
                'Start Pos: '
                '${logMetadata.startPosition.latitude.toStringAsFixed(5)}, '
                '${logMetadata.startPosition.longitude.toStringAsFixed(5)}',
              ),
              const SizedBox(height: 4),
              Text(
                'End Pos: '
                '${logMetadata.endPosition.latitude.toStringAsFixed(5)}, '
                '${logMetadata.endPosition.longitude.toStringAsFixed(5)}',
              ),
            ],
          ),
          IconButton(onPressed: onDelete, icon: Icon(Icons.delete)),
        ],
      ),
    );
  }
}
```

### Utility Functions
```dart
import 'package:flutter/foundation.dart';
import 'package:flutter/services.dart';
import 'package:path_provider/path_provider.dart' as path_provider;
import 'package:path/path.dart' as path;

import 'dart:io';

Future<String> getDirectoryPath(String dirName) async {
  final docDirectory = Platform.isAndroid
      ? await path_provider.getExternalStorageDirectory()
      : await path_provider.getApplicationDocumentsDirectory();

  String absPath = docDirectory!.path;

  final expectedPath = path.joinAll([absPath, "Data", dirName]);
  return expectedPath;
}

//Copy the .gm file from assets directory to app documents directory
Future<void> copyLogToAppDocsDir(String logName) async {
  if (!kIsWeb) {
    final logsDirectory = await getDirectoryPath("Tracks");
    final gpxFile = File('$logsDirectory/$logName');
    final fileBytes = await rootBundle.load('assets/$logName');
    final buffer = fileBytes.buffer;
    await gpxFile.writeAsBytes(
      buffer.asUint8List(fileBytes.offsetInBytes, fileBytes.lengthInBytes),
    );
  }
}
```


