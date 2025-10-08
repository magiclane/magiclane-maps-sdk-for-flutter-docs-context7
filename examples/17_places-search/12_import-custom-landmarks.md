---
description: Documentation for Import Custom Landmarks
title: Import Custom Landmarks
---

# Import Custom Landmarks

In this guide, you will learn how to display a map, import a **LandmarkStore** from a **KML** file, render custom landmarks on the map, **search within those custom landmarks**, and **tap** to view details.

## How it works

This example demonstrates the following key features:

- Display a map.

- Import a `LandmarkStore` from a KML file.

- Display the custom Landmarks on the map.

- Search on custom Landmarks.

- Tap and get info about the custom Landmarks.

This workflow involves initializing a map view, loading landmarks from an external KML file, and adding them to a dedicated `LandmarkStore`.  
Once imported, these landmarks become available for rendering on the map, for targeted searches, and for interaction through user input such as taps or selections.

### UI and Map Integration

The main app shows a map, imports landmarks from a KML file into a `LandmarkStore`, adds that store to map preferences, and enables searching only in the imported store.

When the KML file is imported, the data is parsed and stored in a `LandmarkStore` object.  
The `LandmarkStore` is then registered in the map controller’s preferences so the map engine can display the new landmarks.  
By updating the `SearchPreferences` to include only this store, all subsequent searches will be scoped to these imported landmarks, improving search accuracy and performance.
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
      title: 'Import custom landmarks',
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
  Landmark? _focusedLandmark;
  late SearchPreferences preferences;
  bool isStoreCreated = false;

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
          "Import custom landmarks",
          style: TextStyle(color: Colors.white),
        ),
        actions: [
          IconButton(
            onPressed: addLandmarkStore,
            icon: const Icon(Icons.publish, color: Colors.white),
          ),
          if (isStoreCreated)
            IconButton(
              onPressed: () => _onSearchButtonPressed(context, preferences),
              icon: const Icon(Icons.search, color: Colors.white),
            ),
        ],
      ),
      body: Stack(
        children: [
          GemMap(
            key: ValueKey("GemMap"),
            onMapCreated: _onMapCreated,
            appAuthorization: projectApiToken,
          ),
          if (_focusedLandmark != null)
            Positioned(
              bottom: 30,
              child: LandmarkPanel(
                onCancelTap: _onCancelLandmarkPanelTap,
                landmark: _focusedLandmark!,
              ),
            ),
        ],
      ),
      resizeToAvoidBottomInset: false,
    );
  }

  void _onMapCreated(GemMapController controller) {
    _mapController = controller;
    _registerLandmarkTapCallback();
  }

  Future<int?> _importLandmarks() async {
    final completer = Completer<bool>();

    final file = await assetToUint8List('assets/airports_europe.kml');

    final store = LandmarkStoreService.createLandmarkStore('archies_europe');
    //Ensure the store is empty before importing
    store.removeAllLandmarks();

    final img = await assetToUint8List('assets/KMLCategory.png');

    store.importLandmarksWithDataBuffer(
      buffer: file,
      format: LandmarkFileFormat.kml,
      image: Img(img),
      onComplete: (err) {
        if (err != GemError.success) {
          completer.complete(false);
        } else {
          completer.complete(true);
        }
      },
      categoryId: -1,
    );

    final res = await completer.future;
    if (res) {
      return store.id;
    } else {
      LandmarkStoreService.removeLandmarkStore(store.id);
      throw "Error importing landmarks";
    }
  }

  void addLandmarkStore() async {
    final id = await _importLandmarks();
    final store = LandmarkStoreService.getLandmarkStoreById(id!);

    _mapController.preferences.lmks.add(store!);

    _mapController.centerOnCoordinates(
      Coordinates(latitude: 53.70762, longitude: -1.61112),
      screenPosition: Point(
        _mapController.viewport.width ~/ 2,
        _mapController.viewport.height ~/ 2,
      ),
      zoomLevel: 25,
    );

    // Add the store to the search preferences
    preferences = SearchPreferences();
    preferences.landmarks.add(store);

    // If no results from the map POIs should be returned then searchMapPOIs should be set to false
    preferences.searchMapPOIs = false;

    // If no results from the addresses should be returned then searchAddresses should be set to false
    preferences.searchAddresses = false;

    setState(() {
      isStoreCreated = true;
    });
  }

  Future<void> _registerLandmarkTapCallback() async {
    _mapController.registerOnTouch((pos) async {
      // Select the object at the tap position.
      await _mapController.setCursorScreenPosition(pos);

      // Get the selected landmarks.
      final landmarks = _mapController.cursorSelectionLandmarks();

      // Reset the cursor position back to middle of the screen
      await _mapController.resetMapSelection();

      // Check if there is a selected Landmark.
      if (landmarks.isNotEmpty) {
        _highlightLandmark(landmarks);
        return;
      }

      // Get the selected streets.
      final streets = _mapController.cursorSelectionStreets();

      // Check if there is a selected street.
      if (streets.isNotEmpty) {
        _highlightLandmark(streets);
        return;
      }

      final coordinates = _mapController.transformScreenToWgs(
        Point<int>(pos.x, pos.y),
      );

      // If no landmark was found, we create one.
      final lmk = Landmark.withCoordinates(coordinates);
      lmk.name = '${coordinates.latitude} ${coordinates.longitude}';
      lmk.setImageFromIcon(GemIcon.searchResultsPin);

      _highlightLandmark([lmk]);
    });
  }

  void _highlightLandmark(List<Landmark> landmarks) {
    final settings = HighlightRenderSettings(
      options: {
        HighlightOptions.showLandmark,
        HighlightOptions.showContour,
        HighlightOptions.overlap,
      },
    );
    // Highlight the landmark on the map.
    _mapController.activateHighlight(landmarks, renderSettings: settings);

    final lmk = landmarks[0];
    setState(() {
      _focusedLandmark = lmk;
    });

    _mapController.centerOnCoordinates(
      lmk.coordinates,
      screenPosition: Point(
        _mapController.viewport.width ~/ 2,
        _mapController.viewport.height ~/ 2,
      ),
      zoomLevel: 50,
    );
  }

  void _onCancelLandmarkPanelTap() {
    _mapController.deactivateAllHighlights();

    setState(() {
      _focusedLandmark = null;
    });
  }

  // Custom method for navigating to search screen
  void _onSearchButtonPressed(
    BuildContext context,
    SearchPreferences preferences,
  ) async {
    // Navigating to search screen. The result will be the selected search result(Landmark)
    final result = await Navigator.of(context).push(
      MaterialPageRoute<dynamic>(
        builder: (context) => SearchPage(
          coordinates: Coordinates(latitude: 53.70762, longitude: -1.61112),
          preferences: preferences,
        ),
      ),
    );

    if (result is Landmark) {
      // Activating the highlight
      _mapController.activateHighlight([
        result,
      ], renderSettings: HighlightRenderSettings());

      // Centering the map on the desired coordinates
      _mapController.centerOnCoordinates(result.coordinates, zoomLevel: 70);
    }
  }

  Future<Uint8List> assetToUint8List(String assetPath) async {
    final byteData = await rootBundle.load(assetPath);
    return byteData.buffer.asUint8List();
  }
}
```

### LandmarkPanel

The landmark panel is responsible for presenting detailed information about the currently selected landmark.  
It retrieves and displays key attributes such as name, category, image, and geographic coordinates, and provides controls for closing the panel or taking actions related to the selected landmark.
```dart
class LandmarkPanel extends StatelessWidget {
  final VoidCallback onCancelTap;

  final Landmark landmark;

  const LandmarkPanel({
    super.key,
    required this.onCancelTap,
    required this.landmark,
  });

  @override
  Widget build(BuildContext context) {
    return Container(
      width: MediaQuery.of(context).size.width - 20,
      padding: const EdgeInsets.symmetric(horizontal: 5),
      margin: const EdgeInsets.symmetric(horizontal: 10),
      decoration: BoxDecoration(
        color: Colors.white,
        borderRadius: BorderRadius.circular(15),
      ),
      child: Row(
        children: [
          Container(
            padding: const EdgeInsets.symmetric(horizontal: 10),
            child: landmark.img.isValid
                ? Image.memory(
                    landmark.img.getRenderableImageBytes(size: Size(50, 50))!,
                  )
                : SizedBox(),
          ),
          Row(
            mainAxisAlignment: MainAxisAlignment.spaceBetween,
            children: [
              SizedBox(
                width: MediaQuery.of(context).size.width - 150,
                child: Row(
                  children: [
                    SizedBox(
                      width: MediaQuery.of(context).size.width - 150,
                      child: Column(
                        mainAxisAlignment: MainAxisAlignment.center,
                        crossAxisAlignment: CrossAxisAlignment.start,
                        children: [
                          Text(
                            landmark.name,
                            maxLines: 2,
                            overflow: TextOverflow.ellipsis,
                            style: const TextStyle(
                              color: Colors.black,
                              fontSize: 16,
                              fontWeight: FontWeight.w600,
                            ),
                          ),
                          const SizedBox(height: 5),
                          Text(
                            landmark.categories.isNotEmpty
                                ? landmark.categories.first.name
                                : '',
                            maxLines: 2,
                            overflow: TextOverflow.ellipsis,
                            style: const TextStyle(
                              color: Colors.black,
                              fontSize: 14,
                              fontWeight: FontWeight.w800,
                            ),
                          ),
                          const SizedBox(height: 5),
                          Text(
                            '${landmark.coordinates.latitude.toString()}, ${landmark.coordinates.longitude.toString()}',
                            maxLines: 2,
                            overflow: TextOverflow.visible,
                            style: const TextStyle(
                              color: Colors.black,
                              fontSize: 14,
                              fontWeight: FontWeight.w400,
                            ),
                          ),
                        ],
                      ),
                    ),
                  ],
                ),
              ),
              SizedBox(
                width: 50,
                child: Column(
                  mainAxisAlignment: MainAxisAlignment.start,
                  children: [
                    Align(
                      alignment: Alignment.topRight,
                      child: IconButton(
                        padding: EdgeInsets.zero,
                        onPressed: onCancelTap,
                        icon: const Icon(
                          Icons.cancel,
                          color: Colors.red,
                          size: 30,
                        ),
                      ),
                    ),
                  ],
                ),
              ),
            ],
          ),
        ],
      ),
    );
  }
}
```

### Search Page

The search page component handles querying landmarks using `SearchService`.  
It applies the `SearchPreferences` configured in the main app, ensuring that search results are limited to the imported `LandmarkStore`.  
Search results are presented in a scrollable list, and selecting a result returns it to the calling screen for highlighting and map centering.
```dart
class SearchPage extends StatefulWidget {
  final Coordinates coordinates;
  final SearchPreferences preferences;
  const SearchPage({
    super.key,
    required this.coordinates,
    required this.preferences,
  });

  @override
  State<SearchPage> createState() => _SearchPageState();
}

class _SearchPageState extends State<SearchPage> {
  List<Landmark> landmarks = [];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        automaticallyImplyLeading: true,
        title: const Text(
          "Search Landmarks",
          style: TextStyle(color: Colors.white),
        ),
        backgroundColor: Colors.deepPurple[900],
        foregroundColor: Colors.white,
      ),
      body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(8.0),
            child: TextField(
              onSubmitted: (value) => _onSearchSubmitted(value),
              cursorColor: Colors.deepPurple[900],
              decoration: const InputDecoration(
                hintText: 'Hint: London',
                hintStyle: TextStyle(color: Colors.black),
                focusedBorder: UnderlineInputBorder(
                  borderSide: BorderSide(color: Colors.deepPurple, width: 2.0),
                ),
              ),
            ),
          ),
          Expanded(
            child: ListView.separated(
              padding: EdgeInsets.zero,
              itemCount: landmarks.length,
              controller: ScrollController(),
              separatorBuilder: (context, index) =>
                  const Divider(indent: 50, height: 0),
              itemBuilder: (context, index) {
                final lmk = landmarks.elementAt(index);
                return SearchResultItem(landmark: lmk);
              },
            ),
          ),
        ],
      ),
    );
  }

  void _onSearchSubmitted(String text) {
    search(text, widget.coordinates, preferences: widget.preferences);
  }

  // Search method. Text and coordinates parameters are mandatory, preferences are optional.
  Future<void> search(
    String text,
    Coordinates coordinates, {
    SearchPreferences? preferences,
  }) async {
    Completer<List<Landmark>> completer = Completer<List<Landmark>>();

    // Calling the search method from the sdk.
    // (err, results) - is a callback function that calls when the computing is done.
    // err is an error code, results is a list of landmarks
    SearchService.search(text, coordinates, preferences: preferences, (
      err,
      results,
    ) async {
      // If there is an error or there aren't any results, the method will return an empty list.
      if (err != GemError.success) {
        completer.complete([]);
        return;
      }

      if (!completer.isCompleted) completer.complete(results);
    });

    final result = await completer.future;

    setState(() {
      landmarks = result;
    });
  }
}

// Class for the search results.
class SearchResultItem extends StatefulWidget {
  final Landmark landmark;

  const SearchResultItem({super.key, required this.landmark});

  @override
  State<SearchResultItem> createState() => _SearchResultItemState();
}

class _SearchResultItemState extends State<SearchResultItem> {
  @override
  Widget build(BuildContext context) {
    return ListTile(
      onTap: () => Navigator.of(context).pop(widget.landmark),
      leading: Container(
        padding: const EdgeInsets.all(8),
        child: widget.landmark.img.isValid
            ? Image.memory(
                widget.landmark.img.getRenderableImageBytes(
                  size: Size(50, 50),
                )!,
              )
            : SizedBox(),
      ),
      title: Text(
        widget.landmark.name,
        overflow: TextOverflow.fade,
        style: const TextStyle(
          color: Colors.black,
          fontSize: 14,
          fontWeight: FontWeight.w400,
        ),
        maxLines: 2,
      ),
      subtitle: Text(
        '${getFormattedDistance(widget.landmark)} ${getAddress(widget.landmark)}',
        overflow: TextOverflow.ellipsis,
        style: const TextStyle(
          color: Colors.black,
          fontSize: 14,
          fontWeight: FontWeight.w400,
        ),
      ),
    );
  }
}

String getAddress(Landmark landmark) {
  final addressInfo = landmark.address;
  final street = addressInfo.getField(AddressField.streetName);
  final city = addressInfo.getField(AddressField.city);
  final country = addressInfo.getField(AddressField.country);

  return " ${street ?? ""} ${city ?? ""} ${country ?? ""}";
}

String getFormattedDistance(Landmark landmark) {
  String formattedDistance = '';

  double distance = (landmark.extraInfo.getByKey(PredefinedExtraInfoKey.gmSearchResultDistance) / 1000) as double;
  formattedDistance = "${distance.toStringAsFixed(0)}km";
  return formattedDistance;
}
```

