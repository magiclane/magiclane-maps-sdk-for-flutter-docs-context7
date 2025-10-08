---
description: Documentation for Map Style Update
title: Map Style Update
---

# Map Style Update

This example showcases how to build a Flutter app featuring an interactive map and how to center the camera on a route area with traffic, using the Maps SDK for Flutter.

## Saving Assets

Before running the app, ensure that you save the necessary file (Oldtime style) into the assets directory.

Update your pubspec.yaml file to include these assets:
```yaml
flutter:
  assets:
    - assets/
```

## How it works

The example app demonstrates the following features:

- Display an interactive map.

- View online and offline map styles.

- Update old local styles.

- Download new styles.

- Set a map style.

### UI and Map Integration

The following code builds a UI with a `GemMap` and a map styles page. The user can update local styles or download other ones.
```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  // A init is required to create the assets directory structure where the
  // styles files are located. The SDK needs to be released before copying
  // the old style files into the assets directory.
  await GemKit.initialize(appAuthorization: projectApiToken);
  await GemKit.release();

  // Simulate old styles
  // delete previously existing style and get some old version
  // AS A USER YOU NEVER DO THAT
  await loadOldStyles(rootBundle);

  final autoUpdate = AutoUpdateSettings(
    isAutoUpdateForRoadMapEnabled: true,
    isAutoUpdateForViewStyleHighResEnabled: false,
    isAutoUpdateForViewStyleLowResEnabled: false,
    isAutoUpdateForResourcesEnabled: false,
  );

  await GemKit.initialize(appAuthorization: projectApiToken, autoUpdateSettings: autoUpdate);

  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(debugShowCheckedModeBanner: false, title: 'Map Styles Update', home: MyHomePage());
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key});

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  // GemMapController object used to interact with the map
  late GemMapController _mapController;

  late StylesProvider stylesProvider;

  @override
  void initState() {
    super.initState();
    stylesProvider = StylesProvider.instance;
  }

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
        title: const Text('Map Styles Update', style: TextStyle(color: Colors.white)),
        actions: [
          IconButton(
            onPressed: () => _onMapButtonTap(context),
            icon: const Icon(Icons.map_outlined, color: Colors.white),
          ),
        ],
      ),
      body: GemMap(key: ValueKey("GemMap"), onMapCreated: _onMapCreated),
    );
  }

  void _onMapCreated(GemMapController controller) async {
    _mapController = controller;
  }

  Future<void> _onMapButtonTap(BuildContext context) async {
    // Initialize the styles provider
    await stylesProvider.init();

    final result = await Navigator.push(
      // ignore: use_build_context_synchronously
      context,
      MaterialPageRoute<ContentStoreItem>(builder: (context) => StylesPage(stylesProvider: stylesProvider)),
    );

    if (result != null) {
      // Handle the returned data

      // Wait for the map refresh to complete
      await Future<void>.delayed(Duration(milliseconds: 800));

      // Set selected map style
      _mapController.preferences.setMapStyle(result);
    }
  }
}
```

### Map Styles Page

The code that displays the list of styles is in the `styles_page.dart` file. You can see both online and offline items and to start/cancel the update styles process.
```dart
class StylesPage extends StatefulWidget {
  final StylesProvider stylesProvider;
  const StylesPage({super.key, required this.stylesProvider});

  @override
  State<StylesPage> createState() => _MapStylesUpdatePageState();
}

class _MapStylesUpdatePageState extends State<StylesPage> {
  final stylesList = <ContentStoreItem>[];

  StylesProvider stylesProvider = StylesProvider.instance;

  int? updateProgress;

  @override
  Widget build(BuildContext context) {
    final offlineStyles = StylesProvider.getOfflineStyles();
    return Scaffold(
      appBar: AppBar(
        automaticallyImplyLeading: true,
        backgroundColor: Colors.deepPurple[900],
        foregroundColor: Colors.white,
        title: Row(
          children: [
            const Text("Update", style: TextStyle(color: Colors.white)),
            const SizedBox(width: 10),
            if (updateProgress != null) Expanded(child: ProgressBar(value: updateProgress!)),
            const SizedBox(width: 10),
          ],
        ),
        actions: [
          if (stylesProvider.canUpdateStyles)
            updateProgress != null
                ? GestureDetector(
                    onTap: () {
                      stylesProvider.cancelUpdateStyles();
                    },
                    child: const Text("Cancel"),
                  )
                : IconButton(
                    onPressed: () {
                      showUpdateDialog();
                    },
                    icon: const Icon(Icons.download),
                  ),
        ],
      ),
      body: FutureBuilder<List<ContentStoreItem>?>(
        future: StylesProvider.getOnlineStyles(),
        builder: (context, snapshot) {
          return CustomScrollView(
            slivers: [
              const SliverToBoxAdapter(child: Text("Local: ")),
              SliverList.separated(
                separatorBuilder: (context, index) => const Divider(indent: 20, height: 0),
                itemCount: offlineStyles.length,
                itemBuilder: (context, index) {
                  final styleItem = offlineStyles.elementAt(index);
                  return Padding(
                    padding: const EdgeInsets.all(8.0),
                    child: OfflineItem(
                      styleItem: styleItem,
                      onItemStatusChanged: () {
                        if (mounted) setState(() {});
                      },
                    ),
                  );
                },
              ),
              const SliverToBoxAdapter(child: Text("Online: ")),
              if (snapshot.connectionState == ConnectionState.waiting)
                const SliverToBoxAdapter(child: Center(child: CircularProgressIndicator()))
              else if (snapshot.data == null || snapshot.data!.isEmpty)
                const SliverToBoxAdapter(
                  child: Center(
                    child: Text(
                      'The list of online styles is not available (missing internet connection or expired local content).',
                      textAlign: TextAlign.center,
                    ),
                  ),
                )
              else
                SliverList.separated(
                  itemCount: snapshot.data!.length,
                  separatorBuilder: (context, index) => const Divider(indent: 20, height: 0),
                  itemBuilder: (context, index) {
                    final styleItem = snapshot.data!.elementAt(index);
                    return Padding(
                      padding: const EdgeInsets.all(8.0),
                      child: OnlineItem(
                        styleItem: styleItem,
                        onItemStatusChanged: () {
                          if (mounted) setState(() {});
                        },
                      ),
                    );
                  },
                ),
            ],
          );
        },
      ),
    );
  }

  void showUpdateDialog() {
    showDialog<dynamic>(
      context: context,
      builder: (context) {
        return CustomDialog(
          title: "Update available",
          content:
              "New style update available.\nSize: ${(StylesProvider.computeUpdateSize() / (1024.0 * 1024.0)).toStringAsFixed(2)} MB\nDo you wish to update?",
          positiveButtonText: "Update",
          negativeButtonText: "Later",
          onPositivePressed: () {
            final statusId = stylesProvider.updateStyles(
              onContentUpdaterStatusChanged: onUpdateStatusChanged,
              onContentUpdaterProgressChanged: onUpdateProgressChanged,
            );

            if (statusId != GemError.success) {
              _showMessage("Error updating $statusId");
            }
          },
          onNegativePressed: () {
            Navigator.pop(context);
          },
        );
      },
    );
  }

  void _showMessage(String message) => ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text(message)));

  void onUpdateProgressChanged(int? value) {
    if (mounted) {
      setState(() {
        updateProgress = value;
      });
    }
  }

  void onUpdateStatusChanged(ContentUpdaterStatus status) {
    if (mounted && isReady(status)) {
      showDialog<dynamic>(
        context: context,
        builder: (context) {
          return CustomDialog(
            title: "Update finished",
            content: "The update is done.",
            positiveButtonText: "Ok",
            negativeButtonText: "", // No negative button for this dialog
            onPositivePressed: () {
              //Navigator.pop(context);
            },
            onNegativePressed: () {
              // You can leave this empty or add additional behavior if needed
            },
          );
        },
      );
    }
  }
}

class ProgressBar extends StatelessWidget {
  final int value;

  const ProgressBar({super.key, required this.value});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text("$value%", style: TextStyle(fontSize: 12.0)),
        LinearProgressIndicator(value: value.toDouble() * 0.01, color: Colors.white, backgroundColor: Colors.grey),
      ],
    );
  }
}
```

### Offline and Online Style Items
```dart
class OfflineItem extends StatefulWidget {
  final ContentStoreItem styleItem;

  final void Function() onItemStatusChanged;

  const OfflineItem({super.key, required this.styleItem, required this.onItemStatusChanged});

  @override
  State<OfflineItem> createState() => _OfflineItemState();
}

class _OfflineItemState extends State<OfflineItem> {
  late Version _clientVersion;
  late Version _updateVersion;

  @override
  Widget build(BuildContext context) {
    final styleItem = widget.styleItem;

    bool isOld = styleItem.isUpdatable;
    _clientVersion = styleItem.clientVersion;
    _updateVersion = styleItem.updateVersion;
    return InkWell(
      onTap: () => _onStyleTap(),
      child: Row(
        children: [
          Image.memory(getStyleImage(styleItem, Size(400, 300))!, width: 175, gaplessPlayback: true),
          Expanded(
            child: ListTile(
              title: Text(
                styleItem.name,
                style: const TextStyle(color: Colors.black, fontSize: 16, fontWeight: FontWeight.w600),
              ),
              subtitle: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    "${(styleItem.totalSize / (1024.0 * 1024.0)).toStringAsFixed(2)} MB",
                    style: const TextStyle(color: Colors.black, fontSize: 16),
                  ),
                  Text("Current Version: ${getString(_clientVersion)}"),
                  if (_updateVersion.major != 0 && _updateVersion.minor != 0)
                    Text("New version available: ${getString(_updateVersion)}")
                  else
                    const Text("Version up to date"),
                ],
              ),
              trailing: (isOld) ? const Icon(Icons.warning, color: Colors.orange) : null,
            ),
          ),
        ],
      ),
    );
  }

  // Method that downloads the current style
  void _onStyleTap() {
    final item = widget.styleItem;

    if (item.isUpdatable) return;

    if (item.isCompleted) {
      Navigator.of(context).pop(item);
      return;
    }

    if (getIsDownloadingOrWaiting(item)) {
      // Pause the download.
      item.pauseDownload();
      setState(() {});
    } else {
      // Download the style.
      _startStyleDownload(item);
    }
  }

  void _onStyleDownloadProgressUpdated(int progress) {
    if (mounted) {
      setState(() {
        print('Progress: $progress');
      });
    }
  }

  void _onStyleDownloadFinished(GemError err) {
    widget.onItemStatusChanged();

    // If success, update state
    if (err == GemError.success && mounted) {
      setState(() {});
    }
  }

  void _startStyleDownload(ContentStoreItem styleItem) {
    // Download style
    styleItem.asyncDownload(
      _onStyleDownloadFinished,
      onProgress: _onStyleDownloadProgressUpdated,
      allowChargedNetworks: true,
    );
  }
}
```


```dart
class OnlineItem extends StatefulWidget {
  final ContentStoreItem styleItem;

  final void Function() onItemStatusChanged;

  const OnlineItem({super.key, required this.styleItem, required this.onItemStatusChanged});

  @override
  State<OnlineItem> createState() => _OnlineItemState();
}

class _OnlineItemState extends State<OnlineItem> {
  int _downloadProgress = 0;

  @override
  void initState() {
    super.initState();

    final styleItem = widget.styleItem;

    _downloadProgress = styleItem.downloadProgress;

    // If the style is downloading pause and start downloading again
    // so the progress indicator updates value from callback
    if (getIsDownloadingOrWaiting(styleItem)) {
      final errCode = styleItem.pauseDownload();

      if (errCode == GemError.success) {
        Future.delayed(Duration(seconds: 1), () {
          _startStyleDownload(styleItem);
        });
      } else {
        print("Download pause for item ${styleItem.id} failed with code $errCode");
      }
    }
  }

  @override
  Widget build(BuildContext context) {
    final styleItem = widget.styleItem;

    return InkWell(
      onTap: () => _onStyleTap(),
      child: Row(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Image.memory(getStyleImage(styleItem, Size(400, 300))!, width: 175, gaplessPlayback: true),
          Expanded(
            child: ListTile(
              title: Text(
                maxLines: 5,
                styleItem.name,
                overflow: TextOverflow.ellipsis,
                style: const TextStyle(color: Colors.black, fontSize: 14, fontWeight: FontWeight.w600),
              ),
              subtitle: Text(
                "${(styleItem.totalSize / (1024.0 * 1024.0)).toStringAsFixed(2)} MB",
                style: const TextStyle(color: Colors.black, fontSize: 16),
              ),
              trailing: SizedBox.square(
                dimension: 50,
                child: Builder(
                  builder: (context) {
                    if (styleItem.isCompleted) {
                      return const Icon(Icons.download_done, color: Colors.green);
                    } else if (getIsDownloadingOrWaiting(styleItem)) {
                      return SizedBox(
                        height: 10,
                        child: CircularProgressIndicator(
                          value: _downloadProgress.toDouble() / 100,
                          color: Colors.blue,
                          backgroundColor: Colors.grey.shade300,
                        ),
                      );
                    } else if (styleItem.status == ContentStoreItemStatus.paused) {
                      return const Icon(Icons.pause);
                    }
                    return const SizedBox.shrink();
                  },
                ),
              ),
            ),
          ),
        ],
      ),
    );
  }

  // Method that downloads the current style
  Future<void> _onStyleTap() async {
    final item = widget.styleItem;

    if (item.isCompleted) {
      Navigator.of(context).pop(item);
    }

    if (getIsDownloadingOrWaiting(item)) {
      // Pause the download.
      item.pauseDownload();
      setState(() {});
    } else {
      // Download the style.
      _startStyleDownload(item);
    }
  }

  void _onStyleDownloadProgressUpdated(int progress) {
    if (mounted) {
      setState(() {
        _downloadProgress = progress;
        print('Progress: $progress');
      });
    }
  }

  void _onStyleDownloadFinished(GemError err) {
    widget.onItemStatusChanged();

    // If success, update state
    if (err == GemError.success && mounted) {
      setState(() {});
    }
  }

  void _startStyleDownload(ContentStoreItem styleItem) {
    // Download style
    styleItem.asyncDownload(
      _onStyleDownloadFinished,
      onProgress: _onStyleDownloadProgressUpdated,
      allowChargedNetworks: true,
    );
  }
}
```

### Styles Provider class
```dart
// Singleton class for persisting update related state and logic between instances of StylesPage
class StylesProvider {
  CurrentStylesStatus _currentStylesStatus = CurrentStylesStatus.unknown;

  ContentUpdater? _contentUpdater;
  void Function(int?)? _onContentUpdaterProgressChanged;

  StylesProvider._privateConstructor();
  static final StylesProvider instance = StylesProvider._privateConstructor();

  Future<void> init() {
    final completer = Completer<void>();

    SdkSettings.setAllowInternetConnection(true);

    // Keep track of the new styles status
    SdkSettings.offBoardListener.registerOnWorldwideRoadMapSupportStatus((status) async {
      print("StylesProvider: Styles status updated: $status");
    });

    SdkSettings.offBoardListener.registerOnAvailableContentUpdate((type, status) {
      if (type == ContentType.viewStyleHighRes || type == ContentType.viewStyleLowRes) {
        _currentStylesStatus = CurrentStylesStatus.fromStatus(status);
      }
      if (!completer.isCompleted) {
        completer.complete();
      }
    });

    // Force trying the style update process
    // The user will be notified via onAvailableContentUpdateCallback
    final code = ContentStore.checkForUpdate(ContentType.viewStyleHighRes);
    print("StylesProvider: checkForUpdate resolved with code $code");

    return completer.future;
  }

  CurrentStylesStatus get stylesStatus => _currentStylesStatus;

  bool get isUpToDate => _currentStylesStatus == CurrentStylesStatus.upToDate;

  bool get canUpdateStyles =>
      _currentStylesStatus == CurrentStylesStatus.expiredData || _currentStylesStatus == CurrentStylesStatus.oldData;

  GemError updateStyles({
    void Function(ContentUpdaterStatus)? onContentUpdaterStatusChanged,
    void Function(int?)? onContentUpdaterProgressChanged,
  }) {
    if (_contentUpdater != null) return GemError.inUse;

    final result = ContentStore.createContentUpdater(ContentType.viewStyleHighRes);
    // If successfully created a new content updater
    // or one already exists
    if (result.$2 == GemError.success || result.$2 == GemError.exist) {
      _contentUpdater = result.$1;
      _onContentUpdaterProgressChanged = onContentUpdaterProgressChanged;
      _onContentUpdaterProgressChanged?.call(0);

      // Call the update method
      _contentUpdater!.update(
        true,
        onStatusUpdated: (status) {
          print("StylesProvider: onNotifyStatusChanged with code $status");
          // fully ready - for all old styles the new styles are downloaded
          // partially ready - only a part of the new styles were downloaded because of memory constraints
          if (isReady(status)) {
            // newer styles are downloaded and everything is set to
            // - delete old styles and keep the new ones
            // - update style version to the new version
            final err = _contentUpdater!.apply();
            print("StylesProvider: apply resolved with code ${err.code}");

            if (err == GemError.success) {
              _currentStylesStatus = CurrentStylesStatus.upToDate;
            }

            _onContentUpdaterProgressChanged?.call(null);
            _onContentUpdaterProgressChanged = null;
            _contentUpdater = null;
          }

          onContentUpdaterStatusChanged?.call(status);
        },
        onProgressUpdated: (progress) {
          _onContentUpdaterProgressChanged?.call(progress);
          print('Progress: $progress');
        },
        onComplete: (error) {
          if (error == GemError.success) {
            print('StylesProvider: Successful update');
          } else {
            print('StylesProvider: Update finished with error $error');
          }
        },
      );
    } else {
      print("StylesProvider: There was an error creating the content updater: ${result.$2}");
    }

    return result.$2;
  }

  void cancelUpdateStyles() {
    _contentUpdater?.cancel();

    _onContentUpdaterProgressChanged?.call(null);
    _onContentUpdaterProgressChanged = null;

    _contentUpdater = null;
  }

  // Method to load the online styles list
  static Future<List<ContentStoreItem>> getOnlineStyles() async {
    final stylesListCompleter = Completer<List<ContentStoreItem>>();

    ContentStore.asyncGetStoreContentList(ContentType.viewStyleHighRes, (err, items, isCached) {
      if (err == GemError.success && items.isNotEmpty) {
        stylesListCompleter.complete(items);
      } else {
        stylesListCompleter.complete([]);
      }
    });

    return stylesListCompleter.future;
  }

  // Method to load the downloaded styles list
  static List<ContentStoreItem> getOfflineStyles() {
    final localStyles = ContentStore.getLocalContentList(ContentType.viewStyleHighRes);

    final result = <ContentStoreItem>[];

    for (final style in localStyles) {
      if (style.status == ContentStoreItemStatus.completed) {
        result.add(style);
      }
    }

    return result;
  }

  // Method to compute update size (sum of all style sizes)
  static int computeUpdateSize() {
    final localStyles = ContentStore.getLocalContentList(ContentType.viewStyleHighRes);

    int sum = 0;

    for (final localStyle in localStyles) {
      if (localStyle.isUpdatable && localStyle.status == ContentStoreItemStatus.completed) {
        sum += localStyle.updateSize;
      }
    }

    return sum;
  }
}

enum CurrentStylesStatus {
  expiredData, // more than one version behind
  oldData, // one version behind
  upToDate, // updated
  unknown; // not received any notification yet

  static CurrentStylesStatus fromStatus(ContentStoreStatus status) {
    switch (status) {
      case ContentStoreStatus.expiredData:
        return CurrentStylesStatus.expiredData;
      case ContentStoreStatus.oldData:
        return CurrentStylesStatus.oldData;
      case ContentStoreStatus.upToDate:
        return CurrentStylesStatus.upToDate;
    }
  }
}

String getString(Version version) => '${version.major}.${version.minor}';

// Map style image preview
Uint8List? getStyleImage(ContentStoreItem contentItem, Size? size) =>
    contentItem.imgPreview.getRenderableImageBytes(size: size, format: ImageFileFormat.png);

bool getIsDownloadingOrWaiting(ContentStoreItem contentItem) => [
  ContentStoreItemStatus.downloadQueued,
  ContentStoreItemStatus.downloadRunning,
  ContentStoreItemStatus.downloadWaitingNetwork,
  ContentStoreItemStatus.downloadWaitingFreeNetwork,
  ContentStoreItemStatus.downloadWaitingNetwork,
].contains(contentItem.status);

bool isReady(ContentUpdaterStatus updaterStatus) =>
    updaterStatus == ContentUpdaterStatus.partiallyReady || updaterStatus == ContentUpdaterStatus.fullyReady;

Future<void> loadOldStyles(AssetBundle assetBundle) async {
  const style = 'Basic_1_Oldtime-1_21_656.style';

  final dirPath = await _getDirPath();
  final resFilePath = path.joinAll([dirPath.path, "Data", "SceneRes"]);

  await _deleteAssets(resFilePath, RegExp(r'Basic_1_Oldtime.+\.style'));
  await _loadAsset(assetBundle, style, resFilePath);
}

Future<bool> _loadAsset(AssetBundle assetBundle, String assetName, String destinationDirectoryPath) async {
  final destinationFilePath = path.join(destinationDirectoryPath, assetName);

  File file = File(destinationFilePath);
  if (await file.exists()) {
    return false;
  }

  await file.create();

  final asset = await assetBundle.load('assets/$assetName');
  final buffer = asset.buffer;
  await file.writeAsBytes(buffer.asUint8List(asset.offsetInBytes, asset.lengthInBytes), flush: true);
  print('INFO: Copied asset $destinationFilePath.');

  return true;
}

Future<Directory> _getDirPath() async {
  if (Platform.isAndroid) {
    return (await getExternalStorageDirectory())!;
  } else if (Platform.isIOS) {
    return await getApplicationDocumentsDirectory();
  } else {
    throw Exception('Platform not supported');
  }
}

Future<void> _deleteAssets(String directoryPath, RegExp pattern) async {
  final directory = Directory(directoryPath);

  if (!directory.existsSync()) {
    print('WARNING: Directory $directoryPath not found.');
  }

  for (final file in directory.listSync()) {
    final filename = path.basename(file.path);
    if (pattern.hasMatch(filename)) {
      try {
        print('INFO DELETE ASSETS: deleting file ${file.path}');
        file.deleteSync();
      } catch (e) {
        print('WARNING: Deleting file ${file.path} failed. Reason:\n${e.toString()}.');
      }
    }
  }
}
```


