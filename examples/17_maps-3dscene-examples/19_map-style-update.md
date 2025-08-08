---
description: Documentation for Map Style Update
title: Map Style Update
---

# Map Style Update

This example showcases how to build a Flutter app featuring an interactive map and how to center the camera on a route area with traffic, using the Maps SDK for Flutter.

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
const projectApiToken = String.fromEnvironment('GEM_TOKEN');

void main() {
  // Ensuring that all Flutter bindings are initialized
  WidgetsFlutterBinding.ensureInitialized();

  final autoUpdate = AutoUpdateSettings(
    isAutoUpdateForRoadMapEnabled: true,
    isAutoUpdateForViewStyleHighResEnabled: false,
    isAutoUpdateForViewStyleLowResEnabled: false,
    isAutoUpdateForResourcesEnabled: false,
  );

  GemKit.initialize(
    appAuthorization: projectApiToken,
    autoUpdateSettings: autoUpdate,
  );

  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Map Styles Update',
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
        title: const Text(
          'Map Styles Update',
          style: TextStyle(color: Colors.white),
        ),
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
      MaterialPageRoute<ContentStoreItem>(
        builder:
            (context) => MapStylesUpdatePage(stylesProvider: stylesProvider),
      ),
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
```dart
class MapStylesUpdatePage extends StatefulWidget {
  final StylesProvider stylesProvider;
  const MapStylesUpdatePage({super.key, required this.stylesProvider});

  @override
  State<MapStylesUpdatePage> createState() => _MapStylesUpdatePageState();
}

class _MapStylesUpdatePageState extends State<MapStylesUpdatePage> {
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
            if (updateProgress != null)
              Expanded(child: ProgressBar(value: updateProgress!)),
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
                separatorBuilder:
                    (context, index) => const Divider(indent: 20, height: 0),
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
                const SliverToBoxAdapter(
                  child: Center(child: CircularProgressIndicator()),
                )
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
                  separatorBuilder:
                      (context, index) => const Divider(indent: 20, height: 0),
                  itemBuilder: (context, index) {
                    final styleItem = snapshot.data!.elementAt(index);
                    return Padding(
                      padding: const EdgeInsets.all(8.0),
                      child: StyleItem(
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
              "New world map available.\nSize: ${(StylesProvider.computeUpdateSize() / (1024.0 * 1024.0)).toStringAsFixed(2)} MB\nDo you wish to update?",
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

  void _showMessage(String message) => ScaffoldMessenger.of(
    context,
  ).showSnackBar(SnackBar(content: Text(message)));

  void onUpdateProgressChanged(int? value) {
    if (mounted) {
      setState(() {
        updateProgress = value;
      });
    }
  }

  void onUpdateStatusChanged(ContentUpdaterStatus status) {
    if (mounted && status.isReady) {
      showDialog<dynamic>(
        context: context,
        builder: (context) {
          return CustomDialog(
            title: "Update finished",
            content: "The update is done.",
            positiveButtonText: "Ok",
            negativeButtonText: "", // No negative button for this dialog
            onPositivePressed: () {
              // You can leave this empty or add additional behavior if needed
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
        LinearProgressIndicator(
          value: value.toDouble() * 0.01,
          color: Colors.white,
          backgroundColor: Colors.grey,
        ),
      ],
    );
  }
}

class OfflineItem extends StatefulWidget {
  final ContentStoreItem styleItem;

  final void Function() onItemStatusChanged;

  const OfflineItem({
    super.key,
    required this.styleItem,
    required this.onItemStatusChanged,
  });

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
          Image.memory(
            styleItem.getStyleImage(Size(400, 300))!,
            width: 175,
            gaplessPlayback: true,
          ),
          Expanded(
            child: ListTile(
              title: Text(
                styleItem.name,
                style: const TextStyle(
                  color: Colors.black,
                  fontSize: 16,
                  fontWeight: FontWeight.w600,
                ),
              ),
              subtitle: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    "${(styleItem.totalSize / (1024.0 * 1024.0)).toStringAsFixed(2)} MB",
                    style: const TextStyle(color: Colors.black, fontSize: 16),
                  ),
                  Text("Current Version: ${_clientVersion.str}"),
                  if (_updateVersion.major != 0 && _updateVersion.minor != 0)
                    Text("New version available: ${_updateVersion.str}")
                  else
                    const Text("Version up to date"),
                ],
              ),
              trailing:
                  (isOld)
                      ? const Icon(Icons.warning, color: Colors.orange)
                      : null,
            ),
          ),
        ],
      ),
    );
  }

  // Method that downloads the current map
  Future<void> _onStyleTap() async {
    final item = widget.styleItem;

    if (item.isUpdatable) return;

    if (item.isCompleted) {
      Navigator.of(context).pop(item);
    }

    if (item.isDownloadingOrWaiting) {
      // Pause the download.
      item.pauseDownload();
      setState(() {});
    } else {
      // Download the map.
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
      onProgressCallback: _onStyleDownloadProgressUpdated,
      allowChargedNetworks: true,
    );
  }
}

class StyleItem extends StatefulWidget {
  final ContentStoreItem styleItem;

  final void Function() onItemStatusChanged;

  const StyleItem({
    super.key,
    required this.styleItem,
    required this.onItemStatusChanged,
  });

  @override
  State<StyleItem> createState() => _StyleItemState();
}

class _StyleItemState extends State<StyleItem> {
  int _downloadProgress = 0;

  @override
  void initState() {
    super.initState();

    final styleItem = widget.styleItem;

    _downloadProgress = styleItem.downloadProgress;

    // If the style is downloading pause and start downloading again
    // so the progress indicator updates value from callback
    if (styleItem.isDownloadingOrWaiting) {
      final errCode = styleItem.pauseDownload();

      if (errCode == GemError.success) {
        Future.delayed(Duration(seconds: 1), () {
          _startStyleDownload(styleItem);
        });
      } else {
        print(
          "Download pause for item ${styleItem.id} failed with code $errCode",
        );
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
          Image.memory(
            styleItem.getStyleImage(Size(400, 300))!,
            width: 175,
            gaplessPlayback: true,
          ),
          Expanded(
            child: ListTile(
              title: Text(
                maxLines: 5,
                styleItem.name,
                overflow: TextOverflow.ellipsis,
                style: const TextStyle(
                  color: Colors.black,
                  fontSize: 14,
                  fontWeight: FontWeight.w600,
                ),
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
                      return const Icon(
                        Icons.download_done,
                        color: Colors.green,
                      );
                    } else if (styleItem.isDownloadingOrWaiting) {
                      return SizedBox(
                        height: 10,
                        child: CircularProgressIndicator(
                          value: _downloadProgress.toDouble() / 100,
                          color: Colors.blue,
                          backgroundColor: Colors.grey.shade300,
                        ),
                      );
                    } else if (styleItem.status ==
                        ContentStoreItemStatus.paused) {
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

  // Method that downloads the current map
  Future<void> _onStyleTap() async {
    final item = widget.styleItem;

    if (item.isCompleted) {
      Navigator.of(context).pop(item);
    }

    if (item.isDownloadingOrWaiting) {
      // Pause the download.
      item.pauseDownload();
      setState(() {});
    } else {
      // Download the map.
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
      onProgressCallback: _onStyleDownloadProgressUpdated,
      allowChargedNetworks: true,
    );
  }
}
```

### Styles Provider class
```dart
// Singleton class for persisting update related state and logic between instances of MapsPage
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
    SdkSettings.offBoardListener.registerOnWorldwideRoadMapSupportStatus((
      status,
    ) async {
      print("MapsProvider: Maps status updated: $status");
    });

    SdkSettings.offBoardListener.registerOnAvailableContentUpdate((
      type,
      status,
    ) {
      if (type == ContentType.viewStyleHighRes ||
          type == ContentType.viewStyleLowRes) {
        _currentStylesStatus = CurrentStylesStatus.fromStatus(status);
      }
      if (!completer.isCompleted) {
        completer.complete();
      }
    });

    // // Keep track of the new styles status - deprecated
    // SdkSettings.setAllowConnection(
    //   true,
    //   onWorldwideRoadMapSupportStatusCallback: (status) async {
    //     print("MapsProvider: Maps status updated: $status");
    //   },
    //   onAvailableContentUpdateCallback: (type, status) {
    //     if (type == ContentType.viewStyleHighRes ||
    //         type == ContentType.viewStyleLowRes) {
    //       _currentStylesStatus = CurrentStylesStatus.fromStatus(status);
    //     }
    //     if (!completer.isCompleted) {
    //       completer.complete();
    //     }
    //   },
    // );

    // Force trying the style update process
    // The user will be notified via onAvailableContentUpdateCallback

    final code = ContentStore.checkForUpdate(ContentType.viewStyleHighRes);
    print("MapsProvider: checkForUpdate resolved with code $code");

    return completer.future;
  }

  CurrentStylesStatus get stylesStatus => _currentStylesStatus;

  bool get isUpToDate => _currentStylesStatus == CurrentStylesStatus.upToDate;

  bool get canUpdateStyles =>
      _currentStylesStatus == CurrentStylesStatus.expiredData ||
      _currentStylesStatus == CurrentStylesStatus.oldData;

  GemError updateStyles({
    void Function(ContentUpdaterStatus)? onContentUpdaterStatusChanged,
    void Function(int?)? onContentUpdaterProgressChanged,
  }) {
    if (_contentUpdater != null) return GemError.inUse;

    final result = ContentStore.createContentUpdater(
      ContentType.viewStyleHighRes,
    );
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
          // fully ready - for all old maps the new styles are downloaded
          // partially ready - only a part of the new styles were downloaded because of memory constraints
          if (status.isReady) {
            // newer maps are downloaded and everything is set to
            // - delete old maps and keep the new ones
            // - update map version to the new version
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
      print(
        "StylesProvider: There was an erorr creating the content updater: ${result.$2}",
      );
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

    ContentStore.asyncGetStoreContentList(ContentType.viewStyleHighRes, (
      err,
      items,
      isCached,
    ) {
      if (err == GemError.success && items != null) {
        stylesListCompleter.complete(items);
      } else {
        stylesListCompleter.complete([]);
      }
    });

    return stylesListCompleter.future;
  }

  // Method to load the downloaded styles list
  static List<ContentStoreItem> getOfflineStyles() {
    final localStyles = ContentStore.getLocalContentList(
      ContentType.viewStyleHighRes,
    );

    final result = <ContentStoreItem>[];

    for (final map in localStyles) {
      if (map.status == ContentStoreItemStatus.completed) {
        result.add(map);
      }
    }

    return result;
  }

  // Method to compute update size (sum of all style sizes)
  static int computeUpdateSize() {
    final localStyles = ContentStore.getLocalContentList(
      ContentType.viewStyleHighRes,
    );

    int sum = 0;

    for (final localMap in localStyles) {
      if (localMap.isUpdatable &&
          localMap.status == ContentStoreItemStatus.completed) {
        sum += localMap.updateSize;
      }
    }

    return sum;
  }
}

enum CurrentStylesStatus {
  expiredData, // more than one version behind
  oldData, // one version behind maps
  upToDate, // updated maps
  unknown; // not received any notification yet

  static CurrentStylesStatus fromStatus(MapStatus status) {
    switch (status) {
      case MapStatus.expiredData:
        return CurrentStylesStatus.expiredData;
      case MapStatus.oldData:
        return CurrentStylesStatus.oldData;
      case MapStatus.upToDate:
        return CurrentStylesStatus.upToDate;
    }
  }
}

extension VersionExtension on Version {
  String get str => '$major.$minor';
}

extension ContentStoreItemExtension on ContentStoreItem {
  // Map style image preview
  Uint8List? getStyleImage(Size? size) => imgPreview.getRenderableImageBytes(
        size: size,
        format: ImageFileFormat.png,
      );

  bool get isDownloadingOrWaiting => [
        ContentStoreItemStatus.downloadQueued,
        ContentStoreItemStatus.downloadRunning,
        ContentStoreItemStatus.downloadWaitingNetwork,
        ContentStoreItemStatus.downloadWaitingFreeNetwork,
        ContentStoreItemStatus.downloadWaitingNetwork,
      ].contains(status);
}

extension ContentUpdaterStatusExtension on ContentUpdaterStatus {
  bool get isReady =>
      this == ContentUpdaterStatus.partiallyReady ||
      this == ContentUpdaterStatus.fullyReady;
}
```


