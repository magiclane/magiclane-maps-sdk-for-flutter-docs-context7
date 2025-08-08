---
description: Documentation for Send Debug Info
title: Send Debug Info
---

# Send Debug Info

This example showcases how to build a Flutter app featuring an interactive map and how to share debug information as a .txt file, using the Maps SDK for Flutter.

## How it works

The example app demonstrates the following features:

- Display an interactive map.

- Share debug logs.

### UI and Map Integration

The following code builds a UI with an interactive `GemMap` and a share button.
```dart
// Global instance debug console controller
final controller = DebugConsole.instance;

const projectApiToken = String.fromEnvironment('GEM_TOKEN');

void main() async {
  // Turn on logging
  Debug.logCallObjectMethod = true;
  Debug.logCreateObject = true;
  Debug.logLevel = GemLoggingLevel.all;
  Debug.logListenerMethod = true;

  // Subscribe the DebugConsole to listen to running app
  DebugConsole.listen(() {
    runApp(const MyApp());
  });
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Send Debug Info',
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
        title: const Text(
          'Send Debug Info',
          style: TextStyle(color: Colors.white),
        ),
        actions: [
          IconButton(
            onPressed: _shareLogs,
            icon: const Icon(Icons.share, color: Colors.white),
          ),
        ],
      ),
      body: const GemMap(
        key: ValueKey("GemMap"),
        appAuthorization: projectApiToken,
      ),
    );
  }

  Future<void> _shareLogs() async {
    try {
      final logs = DebugConsole.instance.logs;
      if (logs.isEmpty) {
        ScaffoldMessenger.of(
          context,
        ).showSnackBar(const SnackBar(content: Text('No logs to share')));
        return;
      }

      // Process logs as in DebugConsole's saveToFile
      final processedLogs = List<DebugConsoleLog>.from(logs)
        ..sort((a, b) => a.timestamp.compareTo(b.timestamp));

      final directory = await getTemporaryDirectory();
      final file = File('${directory.path}/debug_logs.txt');
      await file.writeAsString(
        processedLogs.map((log) => log.toString()).join('\n'),
      );

      await Share.shareXFiles([XFile(file.path)], text: 'Debug Logs');
    } catch (e) {
      print('Error sharing logs: $e');
    }
  }
}
```


