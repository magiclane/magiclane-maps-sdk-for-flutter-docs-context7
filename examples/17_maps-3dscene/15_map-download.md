---
description: Documentation for Map Download
title: Map Download
---

# Map Download

This example demonstrates how to list the road maps available on the server for download, how to download a map while indicating the download progress, and how to display the download’s finished status.

## How it works

This example demonstrates the following features:

- List road maps available on the server for download.

- Download a map while displaying download progress.

- Indicate when the download is finished.

### UI and Map Integration 

This code defines the main app structure and how it handles the map’s initialization and the navigation to the map download page.
```dart
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
          'Map Download',
          style: TextStyle(color: Colors.white),
        ),
        actions: [
          IconButton(
            onPressed: () => _onMapButtonTap(context),
            icon: const Icon(
              Icons.map_outlined,
              color: Colors.white,
            ),
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
    SdkSettings.setAllowOffboardServiceOnExtraChargedNetwork(
        ServiceGroupType.contentService, true);
  }

  void _onMapButtonTap(BuildContext context) async {
    Navigator.of(context).push(MaterialPageRoute<dynamic>(
      builder: (context) => const MapsPage(),
    ));
  }
}
```

### Maps page

The MapsPage widget fetches the list of maps available for download and displays them in a scrollable list.
```dart
class MapsPage extends StatefulWidget {
  const MapsPage({super.key});

  @override
  State<MapsPage> createState() => _MapsPageState();
}

class _MapsPageState extends State<MapsPage> {
  final mapsList = <ContentStoreItem>[];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        automaticallyImplyLeading: true,
        foregroundColor: Colors.white,
        title: const Text("Maps List", style: TextStyle(color: Colors.white)),
        backgroundColor: Colors.deepPurple[900],
      ),
      body: FutureBuilder<List<ContentStoreItem>>(
        future: getMaps(),
        builder: (context, snapshot) {
          if (!snapshot.hasData || snapshot.data == null) {
            return const Center(child: CircularProgressIndicator());
          }
          return Scrollbar(
            child: ListView.separated(
              padding: EdgeInsets.zero,
              itemCount: snapshot.data!.length,
              separatorBuilder: (context, index) =>
                  const Divider(indent: 50, height: 0),
              itemBuilder: (context, index) {
                final mapItem = snapshot.data!.elementAt(index);
                return MapsItem(mapItem: mapItem);
              },
            ),
          );
        },
      ),
    );
  }
}
```

### Map item

The MapsItem widget represents each map item in the list, allowing users to download or delete maps and see the download progress. The widget also allows pausing and resuming downloads.
```dart
class MapsItem extends StatefulWidget {
  final ContentStoreItem mapItem;

  const MapsItem({super.key, required this.mapItem});

  @override
  State<MapsItem> createState() => _MapsItemState();
}

class _MapsItemState extends State<MapsItem> {
  ContentStoreItem get mapItem => widget.mapItem;

  @override
  void initState() {
    super.initState();

    restartDownloadIfNecessary(mapItem, _onMapDownloadFinished, onProgress: _onMapDownloadProgressUpdated);
  }

  @override
  Widget build(BuildContext context) {
    return Row(
      children: [
        Expanded(
          child: ListTile(
            onTap: _onTileTap,
            leading: Container(
              padding: const EdgeInsets.all(8),
              width: 50,
              child: getImage(mapItem) != null ? Image.memory(getImage(mapItem)!) : SizedBox(),
            ),
            title: Text(
              mapItem.name,
              style: const TextStyle(color: Colors.black, fontSize: 16, fontWeight: FontWeight.w600),
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
                  } else if (getIsDownloadingOrWaiting(mapItem)) {
                    return SizedBox(
                      height: 10,
                      child: CircularProgressIndicator(
                        value: mapItem.downloadProgress / 100.0,
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
        if (mapItem.isCompleted)
          IconButton(
            onPressed: () {
              if (mapItem.deleteContent() == GemError.success) {
                setState(() {});
              }
            },
            padding: EdgeInsets.zero,
            icon: const Icon(Icons.delete),
          ),
      ],
    );
  }

  void _onTileTap() {
    if (!mapItem.isCompleted) {
      if (getIsDownloadingOrWaiting(mapItem)) {
        _pauseDownload();
      } else {
        _downloadMap();
      }
    }
  }

  void _downloadMap() {
    // Download the map.
    mapItem.asyncDownload(
      _onMapDownloadFinished,
      onProgress: _onMapDownloadProgressUpdated,
      allowChargedNetworks: true,
    );
  }

  void _pauseDownload() {
    // Pause the download.
    mapItem.pauseDownload();
    setState(() {});
  }

  void _onMapDownloadProgressUpdated(int progress) {
    if (mounted) {
      setState(() {});
    }
  }

  void _onMapDownloadFinished(GemError err) {
    // If there is no error, we change the state
    if (mounted && err == GemError.success) {
      setState(() {});
    }
  }
}
```

### Util functions

In the file `utils.dart` there are some helping functions and extensions for:

- getting the list of online maps

- pause and restart a download (a trick to re-wire the UI logic to the `ContentStoreItem`s) - Important to mention is that restarting the download is only made after the item is completely paused.
```dart
bool getIsDownloadingOrWaiting(ContentStoreItem contentItem) => [
  ContentStoreItemStatus.downloadQueued,
  ContentStoreItemStatus.downloadRunning,
  ContentStoreItemStatus.downloadWaitingNetwork,
  ContentStoreItemStatus.downloadWaitingFreeNetwork,
  ContentStoreItemStatus.downloadWaitingNetwork,
].contains(contentItem.status);

// Method that returns the image of the country associated with the road map item
Uint8List? getImage(ContentStoreItem contentItem) {
  Img? img = MapDetails.getCountryFlagImg(contentItem.countryCodes[0]);
  if (img == null) return null;
  if (!img.isValid) return null;
  return img.getRenderableImageBytes(size: Size(100, 100));
}

void restartDownloadIfNecessary(
  ContentStoreItem contentItem,
  void Function(GemError err) onCompleteCallback, {
  void Function(int progress)? onProgress,
}) {
  //If the map is downloading pause and start downloading again
  //so the progress indicator updates value from callback
  if (getIsDownloadingOrWaiting(contentItem)) {
    _pauseAndRestartDownload(contentItem, onCompleteCallback, onProgress: onProgress);
  }
}

void _pauseAndRestartDownload(
  ContentStoreItem contentItem,

  void Function(GemError err) onCompleteCallback, {
  void Function(int progress)? onProgress,
}) {
  final errCode = contentItem.pauseDownload(
    onComplete: (err) {
      if (err == GemError.success) {
        // Download the map.
        contentItem.asyncDownload(
          onCompleteCallback,
          onProgress: onProgress,
          allowChargedNetworks: true,
        );
      } else {
        print("Download pause for item ${contentItem.id} failed with code $err");
      }
    },
  );

  if (errCode != GemError.success) {
    print("Download pause for item ${contentItem.id}  failed with code $errCode");
  }
}

// Method to load the maps
Future<List<ContentStoreItem>> getMaps() async {
  final mapsListCompleter = Completer<List<ContentStoreItem>>();

  ContentStore.asyncGetStoreContentList(ContentType.roadMap, (err, items, isCached) {
    if (err == GemError.success) {
      mapsListCompleter.complete(items);
    } else {
      mapsListCompleter.complete([]);
    }
  });

  return mapsListCompleter.future;
}
```


