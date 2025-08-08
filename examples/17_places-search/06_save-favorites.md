---
description: Documentation for Save Favorites
title: Save Favorites
---

# Save favorites

In this guide, you will learn how to integrate map functionality and save landmarks to a favorites collection.

## How It Works

This example demonstrates the following key features:

- Allows users to save and remove landmarks from their list of favorites.

- Interacts with a map where users can tap on landmarks, check if they are favorites, and toggle them.

### UI and Map Integration

Define the main application widget, MyApp. Create the stateful widget, `MyHomePage`, which will handle the map and favorite locations functionality. Within ``_MyHomePageState``, define the necessary state variables and methods to manage favorites.
```dart
class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key});

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  late GemMapController _mapController;

  Landmark? _focusedLandmark;

  // LandmarkStore object to save Landmarks.
  late LandmarkStore? _favoritesStore;

  bool _isLandmarkFavorite = false;

  final favoritesStoreName = 'Favorites';

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
        title: const Text('Favourites', style: TextStyle(color: Colors.white)),
        actions: [
          IconButton(
            onPressed: () => _onFavouritesButtonPressed(context),
            icon: const Icon(Icons.favorite, color: Colors.white),
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
              bottom: 10,
              child: LandmarkPanel(
                onCancelTap: _onCancelLandmarkPanelTap,
                onFavoritesTap: _onFavoritesLandmarkPanelTap,
                isFavoriteLandmark: _isLandmarkFavorite,
                landmark: _focusedLandmark!,
              ),
            ),
        ],
      ),
      resizeToAvoidBottomInset: false,
    );
  }

  // The callback for when map is ready to use.
  Future<void> _onMapCreated(GemMapController controller) async {
    // Save controller for further usage.
    _mapController = controller;

    // Retrieves the LandmarkStore with the given name.
    _favoritesStore = LandmarkStoreService.getLandmarkStoreByName(
      favoritesStoreName,
    );

    // If there is no LandmarkStore with this name, then create it.
    _favoritesStore ??= LandmarkStoreService.createLandmarkStore(
      favoritesStoreName,
    );

    // Listen for map landmark selection events.
    await _registerLandmarkTapCallback();
  }

```

### Define Landmark Selection and Management

Implement methods to manage landmark selection and favorites.
```dart
Future<void> _registerLandmarkTapCallback() async {
    _mapController.registerTouchCallback((pos) async {
      // Select the object at the tap position.
      await _mapController.setCursorScreenPosition(pos);

      // Get the selected landmarks.
      final landmarks = _mapController.cursorSelectionLandmarks();

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
      zoomLevel: 70,
    );

    _checkIfFavourite();
  }

  // Method to navigate to the Favourites Page.
  void _onFavouritesButtonPressed(BuildContext context) async {
    // Fetch landmarks from the store
    final favoritesList = _favoritesStore!.getLandmarks();

    // Navigating to favorites screen then the result will be the selected item in the list.
    final result = await Navigator.of(context).push(
      MaterialPageRoute<dynamic>(
        builder: (context) => FavoritesPage(landmarkList: favoritesList),
      ),
    );

    if (result is Landmark) {
      // Highlight the landmark on the map.
      _mapController.activateHighlight([
        result,
      ], renderSettings: HighlightRenderSettings());

      // Centering the camera on landmark's coordinates.
      _mapController.centerOnCoordinates(result.coordinates);

      setState(() {
        _focusedLandmark = result;
      });
      _checkIfFavourite();
    }
  }

  void _onCancelLandmarkPanelTap() {
    // Remove landmark highlights from the map.
    _mapController.deactivateAllHighlights();

    setState(() {
      _focusedLandmark = null;
      _isLandmarkFavorite = false;
    });
  }

  void _onFavoritesLandmarkPanelTap() {
    _checkIfFavourite();

    if (_isLandmarkFavorite) {
      // Remove the landmark to the store.
      _favoritesStore!.removeLandmark(_focusedLandmark!);
    } else {
      // Add the landmark to the store.
      _favoritesStore!.addLandmark(_focusedLandmark!);
    }
    setState(() {
      _isLandmarkFavorite = !_isLandmarkFavorite;
    });
  }

  // Utility method to check if the highlighted landmark is favourite.
  void _checkIfFavourite() {
    final focusedLandmarkCoords = _focusedLandmark!.coordinates;
    final favourites = _favoritesStore!.getLandmarks();

    for (final lmk in favourites) {
      late Coordinates coords;
      coords = lmk.coordinates;

      if (focusedLandmarkCoords.latitude == coords.latitude && focusedLandmarkCoords.longitude == coords.longitude) {
        setState(() {
          _isLandmarkFavorite = true;
        });
        return;
      }
    }

    setState(() {
      _isLandmarkFavorite = false;
    });
  }
```

### Favorites Page
```dart
import 'package:gem_kit/core.dart';

import 'package:flutter/material.dart';

class FavoritesPage extends StatefulWidget {
  final List<Landmark> landmarkList;
  const FavoritesPage({super.key, required this.landmarkList});

  @override
  State<FavoritesPage> createState() => _FavoritesPageState();
}

class _FavoritesPageState extends State<FavoritesPage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        foregroundColor: Colors.white,
        automaticallyImplyLeading: true,
        title: const Text("Favorites list"),
        backgroundColor: Colors.deepPurple[900],
      ),
      body: ListView.separated(
        padding: EdgeInsets.zero,
        itemCount: widget.landmarkList.length,
        separatorBuilder:
            (context, index) => const Divider(indent: 50, height: 0),
        itemBuilder: (context, index) {
          final lmk = widget.landmarkList.elementAt(index);
          return FavoritesItem(landmark: lmk);
        },
      ),
    );
  }
}

// Class for favorites landmark.
class FavoritesItem extends StatefulWidget {
  final Landmark landmark;

  const FavoritesItem({super.key, required this.landmark});

  @override
  State<FavoritesItem> createState() => _FavoritesItemState();
}

class _FavoritesItemState extends State<FavoritesItem> {
  @override
  Widget build(BuildContext context) {
    return ListTile(
      onTap: () => Navigator.of(context).pop(widget.landmark),
      leading: Container(
        padding: const EdgeInsets.all(8),
        child: widget.landmark.img.isValid
            ? Image.memory(widget.landmark.img.getRenderableImageBytes(size: Size(50, 50))!)
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
        '${widget.landmark.coordinates.latitude.toString()}, ${widget.landmark.coordinates.longitude.toString()}',
        overflow: TextOverflow.fade,
        style: const TextStyle(
          color: Colors.black,
          fontSize: 14,
          fontWeight: FontWeight.w400,
        ),
        maxLines: 2,
      ),
    );
  }
}
```

### Landmark Panel
```dart
class LandmarkPanel extends StatelessWidget {
  final VoidCallback onCancelTap;
  final VoidCallback onFavoritesTap;

  final bool isFavoriteLandmark;
  final Landmark landmark;

  const LandmarkPanel({
    super.key,
    required this.onCancelTap,
    required this.onFavoritesTap,
    required this.isFavoriteLandmark,
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
            child:
            landmark.img.isValid ? Image.memory(landmark.img.getRenderableImageBytes(size: Size(50, 50))!) : SizedBox(),
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
                    IconButton(
                      padding: EdgeInsets.zero,
                      onPressed: onFavoritesTap,
                      icon: Icon(
                        isFavoriteLandmark
                            ? Icons.favorite
                            : Icons.favorite_outline,
                        color: Colors.red,
                        size: 40,
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


