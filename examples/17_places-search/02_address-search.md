---
description: Documentation for Address Search
title: Address Search
---

# Address Search

This example demonstrates how to create a Flutter app that performs address searches and displays the results on a map using Maps SDK for Flutter.

## How it works

The example app demonstrates the following features:

- Search for a specific address by country, city, street, and house number.

- Highlight and center the map on the searched address.

 

### UI and Map Integration

This code sets up the basic structure of the app, including the map and the app bar. It also provides a search button in the app bar for initiating the address search.
```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Address Search',
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
        title: const Text('Address Search', style: TextStyle(color: Colors.white)),
        actions: [
          IconButton(
              onPressed: () => _onSearchButtonPressed(context).then(
                  (value) => ScaffoldMessenger.of(context).clearSnackBars()),
              icon: const Icon(
                Icons.search,
                color: Colors.white,
              ))
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
```

### Address Search and Map Interaction

This code enables the app to search for addresses by different levels of detail (country, city, street, house number) and highlights the searched location on the map. The map is centered on the found address, with an optional animation.
```dart
Future<void> _onSearchButtonPressed(BuildContext context) async {
  _showSnackBar(context, message: "Search is in progress.");

  // Predefined landmark for Spain.
  final countryLandmark = GuidedAddressSearchService.getCountryLevelItem('ESP');
  print('Country: ${countryLandmark.name}');

  // Use the address search to get a landmark for a city in Spain (e.g., Barcelona).
  final cityLandmark = await _searchAddress(
      landmark: countryLandmark,
      detailLevel: AddressDetailLevel.city,
      text: 'Barcelona');
  if (cityLandmark == null) return;
  print('City: ${cityLandmark.name}');

  // Use the address search to get a predefined street's landmark in the city (e.g., Carrer de Mallorca).
  final streetLandmark = await _searchAddress(
      landmark: cityLandmark,
      detailLevel: AddressDetailLevel.street,
      text: 'Carrer de Mallorca');
  if (streetLandmark == null) return;
  print('Street: ${streetLandmark.name}');

  // Use the address search to get a predefined house number's landmark on the street (e.g., House Number 401).
  final houseNumberLandmark = await _searchAddress(
      landmark: streetLandmark,
      detailLevel: AddressDetailLevel.houseNumber,
      text: '401');
  if (houseNumberLandmark == null) return;
  print('House number: ${houseNumberLandmark.name}');

  // Center the map on the final result.
  _presentLandmark(houseNumberLandmark);
}

void _presentLandmark(Landmark landmark) {
  // Highlight the landmark on the map.
  _mapController.activateHighlight([landmark]);

  // Create an animation (optional).
  final animation = GemAnimation(type: AnimationType.linear);

  // Use the map controller to center on coordinates.
  _mapController.centerOnCoordinates(landmark.coordinates,
      animation: animation);
}

// Address search method.
Future<Landmark?> _searchAddress({
  required Landmark landmark,
  required AddressDetailLevel detailLevel,
  required String text,
}) async {
  final completer = Completer<Landmark?>();

  // Calling the address search SDK method.
  // (err, results) - is a callback function that gets called when the search is finished.
  // err is an error enum, results is a list of landmarks.
  GuidedAddressSearchService.search(text, landmark, detailLevel, (
    err,
    results,
  ) {
    // If there is an error, the method will return a null list.
    if (err != GemError.success && err != GemError.reducedResult ||
        results.isEmpty) {
      completer.complete(null);
      return;
    }

    completer.complete(results.first);
  });

  return completer.future;
}

// Show a snackbar indicating that the search is in progress.
void _showSnackBar(BuildContext context,
    {required String message, Duration duration = const Duration(hours: 1)}) {
  final snackBar = SnackBar(
    content: Text(message),
    duration: duration,
  );

  ScaffoldMessenger.of(context).showSnackBar(snackBar);
}
```


