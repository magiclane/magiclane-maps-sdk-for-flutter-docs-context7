---
description: Documentation for Search Geocoding Features
title: Search Geocoding Features
---

# Search & Geocoding features

This guide explains how to use geocoding and reverse geocoding features to convert coordinates to addresses and vice versa, search along routes, and implement auto-suggestions.

---

## Convert coordinates to addresses

Transform geographic coordinates into detailed address information including country, city, street name, and postal code.

Search around a coordinate to get the corresponding address. The **AddressInfo** object contains information about the country, city, street name, street number, postal code, state, district, and country code.

Access individual fields using the `getField` method, or convert the entire address to a formatted string using the `format` method.
```dart
final SearchPreferences prefs = SearchPreferences(thresholdDistance: 50);
Coordinates coordinates = Coordinates(
    latitude: 51.519305,
    longitude: -0.128022,
);

SearchService.searchAroundPosition(
    coordinates,
    preferences: prefs,
    (err, results) {
    if (err != GemError.success || results.isEmpty) {
        showSnackbar("No results found");
    }
    else {
        Landmark landmark = results.first;
        AddressInfo addressInfo = landmark.address;

        String? country = addressInfo.getField(AddressField.country);
        String? city = addressInfo.getField(AddressField.city);
        String? street = addressInfo.getField(AddressField.streetName);
        String? streetNumber = addressInfo.getField(AddressField.streetNumber);

        String fullAddress = addressInfo.format(includeFields: AddressField.values);
        showSnackbar("Address: $fullAddress");
    }
    },
);
```

---

## Convert addresses to coordinates

Convert address components into geographic coordinates using a hierarchical structure.

Addresses follow a tree-like structure where each node is a **Landmark** with a specific `AddressDetailLevel`. The hierarchy starts with country-level landmarks, followed by cities, streets, and house numbers.

The address structure varies by country. Some countries do not have states or provinces. Use the `getNextAddressDetailLevel` method from the `GuidedAddressSearchService` class to get the next available levels in the address hierarchy.

### Search for countries

Search at the country level to find the parent landmark for hierarchical address searches.
```dart
  GuidedAddressSearchService.searchCountries("Germany", (err, result) {
    if (err != GemError.success && err != GemError.reducedResult) {
      showSnackbar("Error $err");    
    }

    if (result.isEmpty) {
      showSnackbar("No results");
    }

    // do something with "result"
  });
```

This method restricts results to country-level landmarks and works with flexible search terms regardless of language.

### Navigate the address hierarchy

Search through the address structure from country to house number using parent landmarks and detail levels.

Create a function that accepts a parent landmark, an `AddressDetailLevel`, and a text string to return matching child landmarks.

**AddressDetailLevel** values: `noDetail`, `country`, `state`, `county`, `district`, `city`, `settlement`, `postalCode`, `street`, `streetSection`, `streetLane`, `streetAlley`, `houseNumber`, `crossing`.
```dart
// Address search method.
Future<Landmark?> searchAddress({
  required Landmark landmark,
  required AddressDetailLevel detailLevel,
  required String text,
}) async {
  final completer = Completer<Landmark?>();

  GuidedAddressSearchService.search(
    text,
    landmark,
    detailLevel,
    (err, results) {
      // If there is an error, the method will return an empty list.
      if (err != GemError.success && err != GemError.reducedResult ||
          results.isEmpty) {
        completer.complete(null);
        return;
      }

      completer.complete(results.first);
    },
  );

  return completer.future;
}
```

Use this function to search for child landmarks step by step:
```dart
final countryLandmark =
        GuidedAddressSearchService.getCountryLevelItem('ESP');
showSnackbar('Country: ${countryLandmark!.name}');

// Use the address search to get a landmark for a city in Spain (e.g., Barcelona).
final cityLandmark = await searchAddress(
    landmark: countryLandmark,
    detailLevel: AddressDetailLevel.city,
    text: 'Barcelona',
);
if (cityLandmark == null) return;
showSnackbar('City: ${cityLandmark.name}');

// Use the address search to get a predefined street's landmark in the city (e.g., Carrer de Mallorca).
final streetLandmark = await searchAddress(
    landmark: cityLandmark,
    detailLevel: AddressDetailLevel.street,
    text: 'Carrer de Mallorca',
);
if (streetLandmark == null) return;
showSnackbar('Street: ${streetLandmark.name}');

// Use the address search to get a predefined house number's landmark on the street (e.g., House Number 401).
final houseNumberLandmark = await searchAddress(
    landmark: streetLandmark,
    detailLevel: AddressDetailLevel.houseNumber,
    text: '401',
);
if (houseNumberLandmark == null) return;
showSnackbar('House number: ${houseNumberLandmark.name}');
```

The `getCountryLevelItem` method returns the root node for the specified country code. If the country code is invalid, it returns null. Alternatively, use the `searchCountries` method.

---

## Access Wikipedia information

Retrieve Wikipedia content for search results to provide additional context about landmarks.

Perform a standard search, then call `ExternalInfoService.requestWikiInfo` to get Wikipedia descriptions for identified landmarks.

See the [Location Wikipedia guide](../location-wikipedia) for detailed information.

---

## Implement auto-suggestions

Provide real-time search suggestions using Flutter's SearchBar widget.

Call `SearchService.search` with the current text when the search field value changes. This example shows a basic implementation:

**AutoSuggestionSearchWidget** contains the search bar:
```dart
class AutoSuggestionSearchWidget extends StatefulWidget {
  const AutoSuggestionSearchWidget({super.key});

  @override
  State<AutoSuggestionSearchWidget> createState() =>
      _AutoSuggestionSearchWidgetState();
}

class _AutoSuggestionSearchWidgetState
    extends State<AutoSuggestionSearchWidget> {

  TaskHandler? taskHandler;

  @override
  Widget build(BuildContext context) {
    return Center(
      child: SearchAnchor(
        builder: (BuildContext context, SearchController controller) {
          return SearchBar(
            controller: controller,
            onTap: controller.openView,
            onChanged: (_) => controller.openView(),
          );
        },
        suggestionsBuilder:
            (BuildContext context, SearchController controller) async {
          List<Landmark> suggestions = await getAutoSuggestion(controller.text);
          return suggestions.map(
            (lmk) => SearchSuggestion(
              landmark: lmk,
              controller: controller,
            ),
          );
        },
      ),
    );
  }

  Future<List<Landmark>> getAutoSuggestion(String value) async {
    print('New auto suggestion search for $value');

    final Completer<List<Landmark>> completer = Completer();
    final Coordinates refCoordinates = Coordinates(latitude: 48, longitude: 2);
    final SearchPreferences searchPreferences = SearchPreferences(allowFuzzyResults: true);

    // Cancel previous task.
    if (taskHandler != null) {
      SearchService.cancelSearch(taskHandler!);
    }

    // Launch search for new value.
    taskHandler = SearchService.search(
      value,
      refCoordinates,
      preferences: searchPreferences,
      (error, result) {
        completer.complete(result);

        print('Got result for search $value : error - $error, result size - ${result.length}');
      },
    );

    return completer.future;
  }
}
```

Set `allowFuzzyResults` to true for partial match support. Replace `refCoordinates` with the user's current position or map viewport center for better results. Learn more about the SearchBar widget in the [Flutter documentation](https://api.flutter.dev/flutter/material/SearchBar-class.html).

**SearchSuggestion** displays the landmark name and updates the SearchBar text when selected:
```dart
class SearchSuggestion extends StatelessWidget {
  final Landmark landmark;
  final SearchmapController mapController;

  const SearchSuggestion(
      {super.key, required this.landmark, required this.mapController});

  @override
  Widget build(BuildContext context) {
    return ListTile(
      title: Text(landmark.name),
      // Also treat the landmark selection
      onTap: () => mapController.closeView(landmark.name),
    );
  }
}
```

Customize the SearchSuggestion widget to display landmark icons, addresses, or other relevant details based on your use case.

---

## Search along routes

Find landmarks and points of interest along a predefined route.

Use `SearchService.searchAlongRoute` to search for landmarks within a specified distance from the route:
```dart
TaskHandler? taskHandler = SearchService.searchAlongRoute(
    route,
    (err, results) {
        if (err == GemError.success) {
          if (results.isEmpty) {
            showSnackbar("No results");
          } else {
            showSnackbar("Results size: ${results.length}");
            for (final Landmark landmark in results) {
                // do something with landmarks
            }
        }
        } else {
            showSnackbar("No results found");
        }
    },
);
```

Set `SearchPreferences.thresholdDistance` to specify the maximum distance from the route for landmarks to be included. Configure other `SearchPreferences` fields based on your use case.
