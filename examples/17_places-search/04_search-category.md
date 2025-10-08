---
description: Documentation for Search Category
title: Search Category
---

# Search Category

In this guide, you will learn how to integrate map functionality and perform searches for landmarks.

## How it works

This example demonstrates the following key features:

- Search for landmarks based on specific categories.

- Filter landmarks displayed on map by categories, making searches more targeted.

### UI and Map Integration
```dart

void main() async {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Search Category',
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
          "Search Category",
          style: TextStyle(color: Colors.white),
        ),
        actions: [
          IconButton(
            onPressed: () => _onSearchButtonPressed(context),
            icon: const Icon(Icons.search, color: Colors.white),
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

  void _onMapCreated(GemMapController controller) {
    _mapController = controller;
  }

  // Custom method for navigating to search screen
  void _onSearchButtonPressed(BuildContext context) async {
    // Taking the coordinates at the center of the screen as reference coordinates for search.
    final x = MediaQuery.of(context).size.width / 2;
    final y = MediaQuery.of(context).size.height / 2;
    final mapCoords = _mapController.transformScreenToWgs(
      Point<int>(x.toInt(), y.toInt()),
    );

    // Navigating to search screen. The result will be the selected search result(Landmark)
    final result = await Navigator.of(context).push(
      MaterialPageRoute<dynamic>(
        builder: (context) =>
            SearchPage(controller: _mapController, coordinates: mapCoords),
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
}
```

### Define Search Functionality

Implement the SearchPage widget that allows users to search for landmarks.
```dart
class SearchPage extends StatefulWidget {
  final GemMapController controller;
  final Coordinates coordinates;

  // Method to get all the generic categories
  final categories = GenericCategories.categories;

  SearchPage({super.key, required this.controller, required this.coordinates});

  @override
  State<SearchPage> createState() => _SearchPageState();
}

class _SearchPageState extends State<SearchPage> {
  final TextEditingController _textController = TextEditingController();

  List<Landmark> landmarks = [];
  List<LandmarkCategory> selectedCategories = [];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        leading: IconButton(onPressed: _onLeadingPressed, icon: const Icon(CupertinoIcons.arrow_left)),
        title: const Text("Search Category"),
        backgroundColor: Colors.deepPurple[900],
        foregroundColor: Colors.white,
        actions: [
          if (landmarks.isEmpty)
            IconButton(onPressed: () => _onSubmitted(_textController.text), icon: const Icon(Icons.search)),
        ],
      ),
      body: Column(
        children: [
          if (landmarks.isEmpty)
            Padding(
              padding: const EdgeInsets.all(8.0),
              child: TextField(
                controller: _textController,
                cursorColor: Colors.deepPurple[900],
                decoration: const InputDecoration(
                  hintText: 'Enter text',
                  hintStyle: TextStyle(color: Colors.black),
                  focusedBorder: UnderlineInputBorder(borderSide: BorderSide(color: Colors.deepPurple, width: 2.0)),
                ),
              ),
            ),
          if (landmarks.isEmpty)
            Expanded(
              child: ListView.separated(
                padding: EdgeInsets.zero,
                itemCount: widget.categories.length,
                controller: ScrollController(),
                separatorBuilder: (context, index) => const Divider(indent: 50, height: 0),
                itemBuilder: (context, index) {
                  return CategoryItem(
                    onTap: () => _onCategoryTap(index),
                    category: widget.categories[index],
                    categoryIcon: widget.categories[index].img,
                  );
                },
              ),
            ),
          if (landmarks.isNotEmpty)
            Expanded(
              child: ListView.separated(
                padding: EdgeInsets.zero,
                itemCount: landmarks.length,
                controller: ScrollController(),
                separatorBuilder: (context, index) => const Divider(indent: 50, height: 0),
                itemBuilder: (context, index) {
                  final lmk = landmarks.elementAt(index);
                  return SearchResultItem(landmark: lmk);
                },
              ),
            ),
          const SizedBox(height: 5),
        ],
      ),
    );
  }

  int? _isCategorySelected(LandmarkCategory category) {
    for (int index = 0; index < selectedCategories.length; index++) {
      if (category.id == selectedCategories[index].id) {
        return index;
      }
    }
    return null;
  }

  void _onSubmitted(String text) {
    // Setting the preferences so the results are only from the selected categories
    SearchPreferences preferences = SearchPreferences(
      maxMatches: 40,
      allowFuzzyResults: false,
      searchMapPOIs: true,
      searchAddresses: false,
    );

    // Adding in search preferences the selected categories
    for (final category in selectedCategories) {
      preferences.landmarks.addStoreCategoryId(category.landmarkStoreId, category.id);
    }

    search(text, widget.coordinates, preferences);
  }

  late Completer<List<Landmark>> completer;

  // Search method
  Future<void> search(String text, Coordinates coordinates, SearchPreferences preferences) async {
    completer = Completer<List<Landmark>>();

    // Calling the search around position SDK method.
    // (err, results) - is a callback function that calls when the computing is done.
    // err is an error code, results is a list of landmarks
    SearchService.searchAroundPosition(coordinates, preferences: preferences, textFilter: text, (err, results) async {
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

  void _onLeadingPressed() {
    if (landmarks.isNotEmpty) {
      landmarks.clear();
      _textController.clear();
      selectedCategories.clear();
      setState(() {});
      return;
    }
    Navigator.pop(context);
  }

  void _onCategoryTap(int index) {
    int? categoryIndex = _isCategorySelected(widget.categories[index]);
    if (categoryIndex != null) {
      selectedCategories.removeAt(categoryIndex);
    } else {
      selectedCategories.add(widget.categories[index]);
    }
  }
}

// Class for the categories.
class CategoryItem extends StatefulWidget {
  final LandmarkCategory category;
  final Img categoryIcon;
  final VoidCallback onTap;

  const CategoryItem({super.key, required this.category, required this.onTap, required this.categoryIcon});

  @override
  State<CategoryItem> createState() => _CategoryItemState();
}

class _CategoryItemState extends State<CategoryItem> {
  bool _isSelected = false;

  @override
  Widget build(BuildContext context) {
    return ListTile(
      onTap: () {
        widget.onTap();
        setState(() {
          _isSelected = !_isSelected;
        });
      },
      leading: Container(
        padding: const EdgeInsets.all(8),
        child: widget.categoryIcon.isValid
            ? Image.memory(widget.categoryIcon.getRenderableImageBytes(size: Size(50, 50))!, gaplessPlayback: true)
            : SizedBox(),
      ),
      title: Text(
        widget.category.name,
        style: const TextStyle(color: Colors.black, fontSize: 16, fontWeight: FontWeight.w600),
      ),
      trailing: (_isSelected) ? const SizedBox(width: 50, child: Icon(Icons.check, color: Colors.grey)) : null,
    );
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
            ? Image.memory(widget.landmark.img.getRenderableImageBytes(size: Size(50, 50))!)
            : SizedBox(),
      ),
      title: Text(
        widget.landmark.name,
        overflow: TextOverflow.fade,
        style: const TextStyle(color: Colors.black, fontSize: 14, fontWeight: FontWeight.w400),
        maxLines: 2,
      ),
      subtitle: Text(
        "${getFormattedDistance(widget.landmark)} ${getAddress(widget.landmark)}",
        overflow: TextOverflow.ellipsis,
        style: const TextStyle(color: Colors.black, fontSize: 14, fontWeight: FontWeight.w400),
      ),
    );
  }
}

String getAddress(Landmark landmark) {
  final addressInfo = landmark.address;
  final street = addressInfo.getField(AddressField.streetName);
  final city = addressInfo.getField(AddressField.city);
  final country = addressInfo.getField(AddressField.country);

  if (street == null && city == null && country == null) {
    return 'Address not available';
  }

  return " ${street ?? ""} ${city ?? ""} ${country ?? ""}";
}

String getFormattedDistance(Landmark landmark) {
  String formattedDistance = '';

  double distance = (landmark.extraInfo.getByKey(PredefinedExtraInfoKey.gmSearchResultDistance) / 1000) as double;
  formattedDistance = "${distance.toStringAsFixed(0)}km";
  return formattedDistance;
}
```


