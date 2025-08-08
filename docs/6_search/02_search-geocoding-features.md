---
description: Documentation for Search Geocoding Features
title: Search Geocoding Features
---

# Search & Geocoding features

The Maps SDK for Flutter provides geocoding and reverse geocoding capabilities. Key features include:

- Reverse Geocoding: Transform geographic coordinates into comprehensive address details, such as country, city, street name, postal code, and more.

- Geocoding: Locate specific places (e.g., cities, streets, or house numbers) based on address components.

- Route-Based Search: Perform searches along predefined routes to identify landmarks and points of interest.

- Wikipedia Integration: Access Wikipedia descriptions and related information for identified landmarks.

- Auto-Suggestion Implementation: Utilize Flutter's SearchBar widget to dynamically generate search suggestions as users type.

## Reverse geocode coordinates to address

Given a coordinate, we can get the corresponding address by searching around the given location and getting the `AddressInfo` associated with the nearest Landmark found nearby. The AddressInfo provides information about the country, city, street name, street number, postal code, state, district, country code, and other relevant information.

Fields from an ``AddressInfo`` object can be accessed via the ``getField`` method or can be automatically converted to a string containing the address info using the `format` method.
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

## Geocode address to location

We will create the example in two steps. First, we create a function that for a parent landmark, an `AddressDetailLevel` and a text returns the children having the required detail level and matching the text.

The possible values for `AddressDetailLevel` are: noDetail, country, state, county, district, city, settlement, postalCode, street, streetSection, streetLane, streetAlley, houseNumber, crossing.
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

Using the function above, we can look for the children landmarks following the conditions:
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

The `getCountryLevelItem` method returns the root node corresponding to the specified country based on the provided country code. If the country code is invalid, the function will return null.

## Geocode location to Wikipedia

A useful functionality when looking for something, is to obtain the content of a Wikipedia page for a specific text filter. This can be done by a normal search, followed by a call to `ExternalInfoService.requestWikiInfo`.

See the [Location Wikipedia guide](../location-wikipedia) for more info.

## Get auto-suggestions

Auto-suggestions can be implemented by calling ``SearchService.search`` with the text filter set to the current text from the flutter SearchBar widget when the field value is changed. A simple example is demonstrated below:

AutoSuggestionSearchWidget is a widget that contains the search bar:
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

The preferences with ``allowFuzzyResults`` set to true allows for partial match during search. The ``refCoordinates`` might be replaced with a more suitable value, such as the user current position or the map viewport center. More information about the SearchBar widget can be found in the [Flutter documentation](https://api.flutter.dev/flutter/material/SearchBar-class.html).

The ``SearchSuggestion`` is a simple widget that displays the landmark name and completes the SearchBar text with the selected landmark name:
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

The SearchSuggestion widget can be modified to display the icon of the landmark, address and any relevant details depending on the usecase.

## Search along a route

It is possible also to do a search along a route, not based on some coordinates. In this case you can use code like the following:
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

We can set a custom value for ``SearchPreferences.thresholdDistance`` in order to specify the maximum distance to the route for the landmarks to be searched. Other ``SearchPreferences`` fields can be specified depending on the usecase.
