---
description: Documentation for Map Styles
title: Map Styles
---

# Map Styles

This example demonstrates how to create a Flutter app that displays an interactive map with a non-default style using Maps SDK for Flutter.

## How it works

This example demonstrates the following features:

- Display and manage various map styles dynamically.

- Provide smooth transitions between styles while ensuring a seamless user experience.

- Download online map styles.

### Build the main application

Define the main application widget, ``MyApp`` .
```dart
void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Map Styles',
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
        title: const Text('Map Styles', style: TextStyle(color: Colors.white)),
        actions: [
          IconButton(
            onPressed: () async => await _onMapButtonTap(context),
            icon: const Icon(Icons.map_outlined, color: Colors.white),
          ),
        ],
      ),
      body: GemMap(
        key: ValueKey("GemMap"),
        onMapCreated: _onMapCreated,
        appAuthorization: projectApiToken,
      ),
    );
  }

  void _onMapCreated(GemMapController controller) async {
    _mapController = controller;

    SdkSettings.setAllowOffboardServiceOnExtraChargedNetwork(
      ServiceGroupType.contentService,
      true,
    );

    _mapController.registerOnSetMapStyle((styleId, stylePath, viaApi) {
      print("Style updated!");
    });
  }

  Future<void> _onMapButtonTap(BuildContext context) async {
    final result = await Navigator.push(
      context,
      MaterialPageRoute<ContentStoreItem>(
        builder: (context) => MapStylesPage(),
      ),
    );

    if (result != null) {
      // Handle the returned data

      // Wait for the map refresh to complete
      await Future<void>.delayed(Duration(milliseconds: 500));

      // Set selected map style
      _mapController.preferences.setMapStyle(result);
    }
  }
}
```

### Map Styles Page
```dart
class MapStylesPage extends StatefulWidget {
  const MapStylesPage({super.key});

  @override
  State<MapStylesPage> createState() => _MapStylesPageState();
}

class _MapStylesPageState extends State<MapStylesPage> {
  final stylesList = <ContentStoreItem>[];

  StylesProvider stylesProvider = StylesProvider.instance;

  @override
  Widget build(BuildContext context) {
    final offlineStyles = StylesProvider.getOfflineStyles();
    return Scaffold(
      appBar: AppBar(
        automaticallyImplyLeading: true,
        backgroundColor: Colors.deepPurple[900],
        foregroundColor: Colors.white,
        title: Text("Map styles"),
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
                    child: OnlineItem(
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
                  separatorBuilder: (context, index) => const Divider(indent: 50, height: 0),
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
}

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

  // Method that downloads the current map
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
      onProgress: _onStyleDownloadProgressUpdated,
      allowChargedNetworks: true,
    );
  }
}
```

### Map Styles Provider
```dart
class StylesProvider {
  StylesProvider._privateConstructor();
  static final StylesProvider instance = StylesProvider._privateConstructor();

  // Method to load the local-available styles
  static List<ContentStoreItem> getOfflineStyles() {
    final localMaps = ContentStore.getLocalContentList(ContentType.viewStyleHighRes);

    final result = <ContentStoreItem>[];

    for (final map in localMaps) {
      if (map.status == ContentStoreItemStatus.completed) {
        result.add(map);
      }
    }

    return result;
  }

  // Method to load the available styles
  static Future<List<ContentStoreItem>> getOnlineStyles() {
    final completer = Completer<List<ContentStoreItem>>();

    ContentStore.asyncGetStoreContentList(ContentType.viewStyleHighRes, (err, items, isCached) {
      if (err != GemError.success) {
        print("Error while getting styles: ${err.name}");
        return;
      }
      completer.complete(items);
    });

    return completer.future;
  }
}

Uint8List? getStyleImage(ContentStoreItem contentItem, Size? size) =>
    contentItem.imgPreview.getRenderableImageBytes(size: size, format: ImageFileFormat.png);

bool getIsDownloadingOrWaiting(ContentStoreItem contentItem) => [
  ContentStoreItemStatus.downloadQueued,
  ContentStoreItemStatus.downloadRunning,
  ContentStoreItemStatus.downloadWaitingNetwork,
  ContentStoreItemStatus.downloadWaitingFreeNetwork,
  ContentStoreItemStatus.downloadWaitingNetwork,
].contains(contentItem.status);

```


