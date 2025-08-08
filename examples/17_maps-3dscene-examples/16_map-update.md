---
description: Documentation for Map Update
title: Map Update
---

# Map Update

In this guide, you will learn how to download and apply updates to maps. The assets directory includes a world map file and a map for Andorra that are used in this example.

## Saving Assets

Before running the app, ensure that you save the necessary files (world map file and Andorra map) into the assets directory.

Update your pubspec.yaml file to include these assets:
```yaml
flutter:
  assets:
    - assets/
```

## How it Works

This example demonstrates the following features:

- Replace the default map with an older version from the app’s assets to simulate a map update (*this is a hack and you shouldn't do it yourself* - is used here only for demonstration purposes).

- Apply the necessary logic to update the map to the latest version.

- Manage and integrate map update functionality seamlessly within the app’s UI.

### Handle the map update logic

The handling of map update logic is made in the `maps_provider.dart` file.
```dart
// Singleton class for persisting update related state and logic between instances of MapsPage
class MapsProvider {
  CurrentMapsStatus _currentMapsStatus = CurrentMapsStatus.unknown;

  ContentUpdater? _contentUpdater;
  void Function(int?)? _onContentUpdaterProgressChanged;

  MapsProvider._privateConstructor();
  static final MapsProvider instance = MapsProvider._privateConstructor();

  Future<void> init() async {
    // Keep track of the new maps status
    SdkSettings.offBoardListener.registerOnWorldwideRoadMapSupportStatus((
      status,
    ) async {
      print("MapsProvider: Maps status updated: $status");
      _currentMapsStatus = CurrentMapsStatus.fromStatus(status);
    });

    // Force trying the map update process
    // The user will be notified via onWorldwideRoadMapSupportStatusCallback
    final code = ContentStore.checkForUpdate(ContentType.roadMap);
    print("MapsProvider: checkForUpdate resolved with code $code");
  }

  CurrentMapsStatus get mapsStatus => _currentMapsStatus;

  bool get isUpToDate => _currentMapsStatus == CurrentMapsStatus.upToDate;

  bool get canUpdateMaps =>
      _currentMapsStatus == CurrentMapsStatus.expiredData ||
      _currentMapsStatus == CurrentMapsStatus.oldData;

  GemError updateMaps({
    void Function(ContentUpdaterStatus)? onContentUpdaterStatusChanged,
    void Function(int?)? onContentUpdaterProgressChanged,
  }) {
    if (_contentUpdater != null) return GemError.inUse;

    final result = ContentStore.createContentUpdater(ContentType.roadMap);
    // If successfully created a new content updater
    // or one already exists
    if (result.$2 == GemError.success || result.$2 == GemError.exist) {
      _contentUpdater = result.$1;
      _onContentUpdaterProgressChanged = onContentUpdaterProgressChanged;

      // Call the update method
      _contentUpdater!.update(
        true,
        onStatusUpdated: (status) {
          print("MapsProvider: onNotifyStatusChanged with code $status");
          // fully ready - for all old maps the new maps are downloaded
          // partially ready - only a part of the new maps were downloaded because of memory constraints
          if (status.isReady) {
            // newer maps are downloaded and everything is set to
            // - delete old maps and keep the new ones
            // - update map version to the new version
            final err = _contentUpdater!.apply();
            print("MapsProvider: apply resolved with code ${err.code}");

            if (err == GemError.success) {
              _currentMapsStatus = CurrentMapsStatus.upToDate;
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
            print('MapsProvider: Successful update');
          } else {
            print('MapsProvider: Update finished with error $error');
          }
        },
      );
    } else {
      print(
        "MapsProvider: There was an erorr creating the content updater: ${result.$2}",
      );
    }

    return result.$2;
  }

  void cancelUpdateMaps() {
    _contentUpdater?.cancel();

    _onContentUpdaterProgressChanged?.call(null);
    _onContentUpdaterProgressChanged = null;

    _contentUpdater = null;
  }

  // Method to load the online map list
  static Future<List<ContentStoreItem>> getOnlineMaps() async {
    final mapsListCompleter = Completer<List<ContentStoreItem>>();

    ContentStore.asyncGetStoreContentList(ContentType.roadMap, (
      err,
      items,
      isCached,
    ) {
      if (err == GemError.success && items != null) {
        mapsListCompleter.complete(items);
      } else {
        mapsListCompleter.complete([]);
      }
    });

    return mapsListCompleter.future;
  }

  // Method to load the downloaded map list
  static List<ContentStoreItem> getOfflineMaps() {
    final localMaps = ContentStore.getLocalContentList(ContentType.roadMap);

    final result = <ContentStoreItem>[];

    for (final map in localMaps) {
      if (map.status == ContentStoreItemStatus.completed) {
        result.add(map);
      }
    }

    return result;
  }

  // Method to compute update size (sum of all maps sizes)
  static int computeUpdateSize() {
    final localMaps = ContentStore.getLocalContentList(ContentType.roadMap);

    int sum = 0;

    for (final localMap in localMaps) {
      if (localMap.isUpdatable &&
          localMap.status == ContentStoreItemStatus.completed) {
        sum += localMap.updateSize;
      }
    }

    return sum;
  }
}

Future<void> loadOldMaps(AssetBundle assetBundle) async {
  const cmap = 'AndorraOSM_2021Q1.cmap';
  const worldMap = 'WM_7_406.map';

  final dirPath = await _getDirPath();
  final resFilePath = path.joinAll([dirPath.path, "Data", "Res"]);
  final mapsFilePath = path.joinAll([dirPath.path, "Data", "Maps"]);

  await _deleteAssets(resFilePath, RegExp(r'WM_\d_\d+\.map'));
  await _deleteAssets(mapsFilePath, RegExp(r'.+\.cmap'));

  await _loadAsset(assetBundle, cmap, mapsFilePath);
  await _loadAsset(assetBundle, worldMap, resFilePath);
}

Future<bool> _loadAsset(
  AssetBundle assetBundle,
  String assetName,
  String destinationDirectoryPath,
) async {
  final destinationFilePath = path.join(destinationDirectoryPath, assetName);

  File file = File(destinationFilePath);
  if (await file.exists()) {
    return false;
  }

  await file.create();

  final asset = await assetBundle.load('assets/$assetName');
  final buffer = asset.buffer;
  await file.writeAsBytes(
    buffer.asUint8List(asset.offsetInBytes, asset.lengthInBytes),
    flush: true,
  );

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
    print(
      '\x1B[31mWARNING: Directory $directoryPath not found. Test might fail.\x1B[0m',
    );
  }

  for (final file in directory.listSync()) {
    final filename = path.basename(file.path);
    if (pattern.hasMatch(filename)) {
      try {
        //print('INFO DELETE ASSETS: deleting file ${file.path}');
        file.deleteSync();
      } catch (e) {
        print(
          '\x1B[31mWARNING: Deleting file ${file.path} failed. Test might fail. Reason:\n${e.toString()}\x1B[0m',
        );
      }
    }
  }
}

enum CurrentMapsStatus {
  expiredData, // more than one version behind
  oldData, // one version behind maps
  upToDate, // updated maps
  unknown; // not received any notification yet

  static CurrentMapsStatus fromStatus(MapStatus status) {
    switch (status) {
      case MapStatus.expiredData:
        return CurrentMapsStatus.expiredData;
      case MapStatus.oldData:
        return CurrentMapsStatus.oldData;
      case MapStatus.upToDate:
        return CurrentMapsStatus.upToDate;
    }
  }
}

extension VersionExtension on Version {
  String get str => '$major.$minor';
}

extension ContentStoreItemExtension on ContentStoreItem {
  // Method that returns the image of the country associated with the road map item
  Uint8List? get image {
    Img? img = MapDetails.getCountryFlagImg(countryCodes[0]);
    if (img == null) return null;
    if (!img.isValid) return null;
    return img.getRenderableImageBytes(size: Size(100, 100));
  }

  bool get isDownloadingOrWaiting => [
        ContentStoreItemStatus.downloadQueued,
        ContentStoreItemStatus.downloadRunning,
        ContentStoreItemStatus.downloadWaitingNetwork,
        ContentStoreItemStatus.downloadWaitingFreeNetwork,
        ContentStoreItemStatus.downloadWaitingNetwork,
      ].contains(status);

  void restartDownloadIfNecessary(
    void Function(GemError err) onCompleteCallback, {
    void Function(int progress)? onProgressCallback,
  }) {
    //If the map is downloading pause and start downloading again
    //so the progress indicator updates value from callback
    if (isDownloadingOrWaiting) {
      _pauseAndRestartDownload(
        onCompleteCallback,
        onProgressCallback: onProgressCallback,
      );
    }
  }

  void _pauseAndRestartDownload(
    void Function(GemError err) onCompleteCallback, {
    void Function(int progress)? onProgressCallback,
  }) {
    final errCode = pauseDownload(
      onComplete: (err) {
        if (err == GemError.success) {
          // Download the map.
          asyncDownload(
            onCompleteCallback,
            onProgressCallback: onProgressCallback,
            allowChargedNetworks: true,
          );
        } else {
          print("Download pause for item $id failed with code $err");
        }
      },
    );

    if (errCode != GemError.success) {
      print("Download pause for item $id failed with code $errCode");
    }
  }
}

extension ContentUpdaterStatusExtension on ContentUpdaterStatus {
  bool get isReady =>
      this == ContentUpdaterStatus.partiallyReady ||
      this == ContentUpdaterStatus.fullyReady;
}
```

This file contains the `MapsProvider` class and a few helpful methods and extensions.

The methods of the `MapsProvider` class do the following:
<table>
<tr><th>Method</th><th>Explanation</th></tr>
<tr>
<td>init()</td>
<td>
<ul>
  <li>initiates the connection and listens to status updates</li>
  <li>forces checking for updates, when we have an answer you can get the map status</li>
</ul>
</td>
</tr>
<tr>
<td>mapStatus</td>
<td>
<ul>
<li>unknown - no answer from the server yet</li>
<li>expired - 2 or more map versions behind</li>
<li>oldData - previous version</li>
<li>upToDate - newest map available</li>
</ul> 
</td>
</tr>
<tr>
<td>updateMaps()</td>
<td>Method that initiates the map update. It has callbacks that notify about progress.</td>
</tr>
<tr>
<td>cancelUpdateMaps()</td>
<td>Method that cancels the map update.</td>
</tr>
<tr>
<td>getOnlineMaps()</td>
<td>Method returning the online maps.</td>
</tr>
<tr>
<td>getOfflineMaps()</td>
<td>Method returning the offline maps on the device.</td>
</tr>
<tr>
<td>computeUpdateSize()</td>
<td>Method returning what would be the size of an update (sum of sizes for offline maps)</td>
</tr>
</table>

There are also some helping enums, extensions and useful methods that allow working with assets.

### The main page with the map

In the `main.dart` file we have the code that displays the map and allows you to tap the button that shows you the maps.
```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  // Simulate old maps
  // delete all maps, all resources and get some old ones
  // AS A USER YOU NEVER DO THAT
  await loadOldMaps(rootBundle);

  final autoUpdate = AutoUpdateSettings(
    isAutoUpdateForRoadMapEnabled: false,
    isAutoUpdateForViewStyleHighResEnabled: false,
    isAutoUpdateForViewStyleLowResEnabled: false,
    isAutoUpdateForHumanVoiceEnabled: false, // default
    isAutoUpdateForComputerVoiceEnabled: false, // default
    isAutoUpdateForCarModelEnabled: false, // default
    isAutoUpdateForResourcesEnabled: false,
  );

  await GemKit.initialize(appAuthorization: projectApiToken, autoUpdateSettings: autoUpdate);
  await MapsProvider.instance.init();

  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(debugShowCheckedModeBanner: false, title: 'Map Update', home: MyHomePage());
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key});
  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  GemMapController? mapController;

  void onMapCreated(GemMapController controller) async {
    mapController = controller;
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
        title: const Text('Map Update', style: TextStyle(color: Colors.white)),
        actions: [
          IconButton(
            onPressed: () => _onMapButtonTap(context),
            icon: const Icon(Icons.map_outlined, color: Colors.white),
          ),
        ],
      ),
      body: GemMap(key: ValueKey("GemMap"), onMapCreated: onMapCreated, appAuthorization: projectApiToken),
      // body: Container(),
    );
  }

  // Method to navigate to the Maps Page.
  void _onMapButtonTap(BuildContext context) async {
    if (mapController != null) {
      Navigator.of(context).push(MaterialPageRoute<dynamic>(builder: (context) => MapsPage()));
    }
  }
}
```

### Displaying the list of maps

The code that displays the list of maps is in the `maps_page.dart` file. You can see both online and offline maps and to start/cancel the update maps process.
```dart
class MapsPage extends StatefulWidget {
  const MapsPage({super.key});

  @override
  State<MapsPage> createState() => _MapsPageState();
}

class _MapsPageState extends State<MapsPage> {
  final mapsList = <ContentStoreItem>[];

  MapsProvider mapsProvider = MapsProvider.instance;
  int? updateProgress;

  @override
  Widget build(BuildContext context) {
    final localMaps = MapsProvider.getOfflineMaps();
    return Scaffold(
      appBar: AppBar(
        automaticallyImplyLeading: true,
        foregroundColor: Colors.white,
        title: Row(
          children: [
            const Text("Maps", style: TextStyle(color: Colors.white)),
            const SizedBox(width: 10),
            if (updateProgress != null)
              Expanded(child: ProgressBar(value: updateProgress!)),
            const SizedBox(width: 10),
          ],
        ),
        actions: [
          if (mapsProvider.canUpdateMaps)
            updateProgress != null
                ? GestureDetector(
                    onTap: () {
                      mapsProvider.cancelUpdateMaps();
                    },
                    child: const Text("Cancel Update"),
                  )
                : IconButton(
                    onPressed: () {
                      showUpdateDialog();
                    },
                    icon: const Icon(Icons.download),
                  ),
        ],
        backgroundColor: Colors.deepPurple[900],
      ),
      body: FutureBuilder<List<ContentStoreItem>?>(
        future: MapsProvider.getOnlineMaps(),
        builder: (context, snapshot) {
          //The CustomScrollView is required in order to render the online map list items lazily

          return CustomScrollView(
            slivers: [
              const SliverToBoxAdapter(child: Text("Local: ")),
              SliverList.separated(
                separatorBuilder: (context, index) =>
                    const Divider(indent: 50, height: 0),
                itemCount: localMaps.length,
                itemBuilder: (context, index) {
                  final mapItem = localMaps.elementAt(index);
                  return OfflineItem(
                      mapItem: mapItem,
                      deleteMap: (map) {
                        if (map.deleteContent() == GemError.success) {
                          setState(() {});
                        }
                      });
                },
              ),
              const SliverToBoxAdapter(child: SizedBox(height: 30)),
              const SliverToBoxAdapter(child: Text("All: ")),
              if (snapshot.connectionState == ConnectionState.waiting)
                const SliverToBoxAdapter(
                  child: Center(child: CircularProgressIndicator()),
                )
              else if (snapshot.data == null || snapshot.data!.isEmpty)
                const SliverToBoxAdapter(
                  child: Center(
                    child: Text(
                      'The list of online maps is not available (missing internet connection or expired local content).',
                      textAlign: TextAlign.center,
                    ),
                  ),
                )
              else
                SliverList.separated(
                  itemCount: snapshot.data!.length,
                  separatorBuilder: (context, index) =>
                      const Divider(indent: 50, height: 0),
                  itemBuilder: (context, index) {
                    final mapItem = snapshot.data!.elementAt(index);
                    return OnlineItem(
                      mapItem: mapItem,
                      onItemStatusChanged: () {
                        if (mounted) setState(() {});
                      },
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
              "New world map available.\nSize: ${(MapsProvider.computeUpdateSize() / (1024.0 * 1024.0)).toStringAsFixed(2)} MB\nDo you wish to update?",
          positiveButtonText: "Update",
          negativeButtonText: "Later",
          onPositivePressed: () {
            final statusId = mapsProvider.updateMaps(
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

  void _showMessage(String message) =>
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text(message)),
      );

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
        Text("$value%"),
        LinearProgressIndicator(
          value: value.toDouble() * 0.01,
          color: Colors.white,
          backgroundColor: Colors.grey,
        ),
      ],
    );
  }
}
```

### Displaying each online item

The code that displays each item is the following:
```dart
class OnlineItem extends StatefulWidget {
  final ContentStoreItem mapItem;

  final void Function() onItemStatusChanged;

  const OnlineItem({
    super.key,
    required this.mapItem,
    required this.onItemStatusChanged,
  });

  @override
  State<OnlineItem> createState() => _OnlineItemState();
}

class _OnlineItemState extends State<OnlineItem> {
  int _downloadProgress = 0;

  ContentStoreItem get mapItem => widget.mapItem;

  @override
  void initState() {
    super.initState();

    _downloadProgress = mapItem.downloadProgress;

    mapItem.restartDownloadIfNecessary(
      _onMapDownloadFinished,
      onProgressCallback: _onMapDownloadProgressUpdated,
    );
  }

  @override
  Widget build(BuildContext context) {
    return Row(
      children: [
        Expanded(
          child: ListTile(
            onTap: () => _onTileTap(),
            leading: Container(
              padding: const EdgeInsets.all(8),
              width: 50,
              child: mapItem.getMapImage(Size(100, 100)) != null
                  ? Image.memory(
                      mapItem.getMapImage(Size(100, 100))!,
                      gaplessPlayback: true,
                    )
                  : SizedBox(),
            ),
            title: Text(
              mapItem.name,
              style: const TextStyle(
                color: Colors.black,
                fontSize: 16,
                fontWeight: FontWeight.w600,
              ),
            ),
            subtitle: Text(
              "${(mapItem.totalSize / (1024.0 * 1024.0)).toStringAsFixed(2)} MB",
              style: const TextStyle(color: Colors.black, fontSize: 16),
            ),
            trailing: SizedBox.square(
              dimension: 50,
              child: Builder(
                builder: (context) {
                  if (mapItem.isCompleted) {
                    return const Icon(Icons.download_done, color: Colors.green);
                  } else if (mapItem.isDownloadingOrWaiting) {
                    return SizedBox(
                      height: 10,
                      child: CircularProgressIndicator(
                        value: _downloadProgress.toDouble() / 100,
                        color: Colors.blue,
                        backgroundColor: Colors.grey.shade300,
                      ),
                    );
                  } else if (mapItem.status == ContentStoreItemStatus.paused) {
                    return const Icon(Icons.pause);
                  }
                  return const SizedBox.shrink();
                },
              ),
            ),
          ),
        ),
        if (_downloadProgress != 0 && !mapItem.isCompleted)
          IconButton(
            onPressed: () {
              if (mapItem.deleteContent() == GemError.success) {
                widget.onItemStatusChanged();
              }
            },
            padding: EdgeInsets.zero,
            icon: const Icon(Icons.delete),
          ),
      ],
    );
  }

  // Method that downloads the current map
  Future<void> _onTileTap() async {
    if (mapItem.isCompleted) return;

    if (mapItem.isDownloadingOrWaiting) {
      // Pause the download.
      mapItem.pauseDownload();
      setState(() {});
    } else {
      // Download the map.
      _startMapDownload(mapItem);
    }
  }

  void _startMapDownload(ContentStoreItem mapItem) {
    mapItem.asyncDownload(
      _onMapDownloadFinished,
      onProgressCallback: _onMapDownloadProgressUpdated,
      allowChargedNetworks: true,
    );
  }

  void _onMapDownloadProgressUpdated(int progress) {
    if (mounted) {
      setState(() {
        _downloadProgress = progress;
        print('Progress: $progress');
      });
    }
  }

  void _onMapDownloadFinished(GemError err) {
    widget.onItemStatusChanged();

    // If success, update state
    if (err == GemError.success && mounted) {
      setState(() {});
    }
  }
}
```

### Displaying each offline item

Each offline item is represented by an instance of the `MapsDownloadedItem` class. The code is the following:
```dart
class OfflineItem extends StatefulWidget {
  final ContentStoreItem mapItem;
  final void Function(ContentStoreItem) deleteMap;

  const OfflineItem({
    super.key,
    required this.mapItem,
    required this.deleteMap,
  });

  @override
  State<OfflineItem> createState() => _OfflineItemState();
}

class _OfflineItemState extends State<OfflineItem> {
  late Version _clientVersion;
  late Version _updateVersion;

  @override
  Widget build(BuildContext context) {
    final mapItem = widget.mapItem;

    bool isOld = mapItem.isUpdatable;
    _clientVersion = mapItem.clientVersion;
    _updateVersion = mapItem.updateVersion;
    return Row(
      children: [
        Expanded(
          child: ListTile(
            leading: Container(
              padding: const EdgeInsets.all(8),
              width: 50,
              child: mapItem.getMapImage(Size(100, 100)) != null
                  ? Image.memory(
                      mapItem.getMapImage(Size(100, 100))!,
                      gaplessPlayback: true,
                    )
                  : SizedBox(),
            ),
            title: Text(
              mapItem.name,
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
                  "${(mapItem.totalSize / (1024.0 * 1024.0)).toStringAsFixed(2)} MB",
                  style: const TextStyle(color: Colors.black, fontSize: 16),
                ),
                Text(
                  "Current Version: ${_clientVersion.str}",
                ),
                if (_updateVersion.major != 0 && _updateVersion.minor != 0)
                  Text(
                    "New version available: ${_updateVersion.str}",
                  )
                else
                  const Text("Version up to date"),
              ],
            ),
            trailing: (isOld)
                ? const Icon(Icons.warning, color: Colors.orange)
                : null,
          ),
        ),
        IconButton(
          onPressed: () => widget.deleteMap(mapItem),
          padding: EdgeInsets.zero,
          icon: const Icon(Icons.delete),
        ),
      ],
    );
  }
}
```


