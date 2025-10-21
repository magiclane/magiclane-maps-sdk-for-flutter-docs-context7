---
description: Documentation for Get Started Search
title: Get Started Search
---

# Getting started with Search

The Maps SDK for Flutter provides a flexible and robust search functionality, allowing the search of locations using text queries and coordinates:

- Text Search: Perform searches using a text query and geographic coordinates to prioritize results within a specific area.

- Search Preferences: Customize search behavior using various options, such as allowing fuzzy results, limiting search distance, or specifying the number of results.

- Category-Based Search: Filter search results by predefined categories, such as gas stations or parking areas.

- Proximity Search: Retrieve all nearby landmarks without specifying a text query.

## Text search

The simplest way to search for something is by providing text and specifying coordinates. The coordinates serve as a hint, prioritizing points of interest (POIs) within the indicated area.
```dart
const text = "Paris";
final coords = Coordinates(latitude: 45, longitude: 10);
final preferences = SearchPreferences(
  maxMatches: 40,
  allowFuzzyResults: true,
);

TaskHandler? taskHandler = SearchService.search(
  text,
  coords,
  preferences: preferences,
  (err, results) async {
    // If there is an error or there aren't any results, the method will return an empty list.
    if (err == GemError.success) {
      if (results.isEmpty) {
        showSnackbar("No results");
      } else {
        showSnackbar("Number of results: ${results.length}");
      }
    } else {
      showSnackbar("Error: $err");
    }
  },
);
```

The `SearchService.search` method returns `null` only when the geographic search fails to initialize. In such cases, calling `SearchService.cancelSearch(taskHandler)` is not possible. Error details will be delivered through the `onComplete` function of the `SearchService.search` method.

The `err` provided by the callback function can have the following values:
<table>
<tr>
<th>Value</th>
<th>Significance</th>
</tr>
<tr>
<td>`GemError.success`</td>
<td>successfully completed</td>
</tr>
<tr>
<td>`GemError.cancel`</td>
<td>cancelled by the user</td>
</tr>
<tr>
<td>`GemError.noMemory`</td>
<td>search engine couldn't allocate the necessary memory for the operation</td>
</tr>
<tr>
<td>`GemError.operationTimeout`</td>
<td>search was executed on the online service and the operation took too much time to complete (usually more than 1 min, depending on the server overload state)</td>
</tr>
<tr>
<td>`GemError.networkTimeout`</td>
<td>can't establish the connection or the server didn't respond on time</td>
</tr>
<tr>
<td>`GemError.networkFailed`</td>
<td>search was executed on the online service and the operation failed due to bad network connection</td>
</tr>
</table>

## Specifying preferences

As seen in the previous example, before searching we need to specify some `SearchPreferences`. The following characteristics apply to a search:

<table>
  <tr>
    <th>Field</th>
    <th>Type</th>
    <th>Default Value</th>
    <th>Explanation</th>
  </tr>
  <tr>
    <td>allowFuzzyResults</td>
    <td>bool</td>
    <td>true</td>
    <td>Allows fuzzy search results, enabling approximate matches for queries.</td>
  </tr>
  <tr>
    <td>estimateMissingHouseNumbers</td>
    <td>bool</td>
    <td>true</td>
    <td>Enables estimation of missing house numbers in address searches.</td>
  </tr>
  <tr>
    <td>exactMatch</td>
    <td>bool</td>
    <td>false</td>
    <td>Restricts results to only those that exactly match the query.</td>
  </tr>
  <tr>
    <td>maxMatches</td>
    <td>int</td>
    <td>40</td>
    <td>Specifies the maximum number of search results to return.</td>
  </tr>
  <tr>
    <td>searchAddresses</td>
    <td>bool</td>
    <td>true</td>
    <td>Includes addresses in the search results. This option also includes roads.</td>
  </tr>
  <tr>
    <td>searchMapPOIs</td>
    <td>bool</td>
    <td>true</td>
    <td>Includes points of interest (POIs) on the map in the search results.</td>
  </tr>
  <tr>
    <td>searchOnlyOnboard</td>
    <td>bool</td>
    <td>false</td>
    <td>Limits the search to onboard (offline) data only.</td>
  </tr>
  <tr>
    <td>thresholdDistance</td>
    <td>int</td>
    <td>2147483647</td>
    <td>Defines the maximum distance (in meters) for search results from the query location.</td>
  </tr>
  <tr>
    <td>easyAccessOnlyResults</td>
    <td>bool</td>
    <td>false</td>
    <td>Restricts results to locations that are easily accessible.</td>
  </tr>
</table>

### Search by category

The Maps SDK for Flutter allows the user to filter results based on the category.
Some predefined categories are available and can be accessed using `GenericCategories.categories`.

In the following example, we perform a search for a text query, limiting the results to the first two categories (e.g., gas stations and parking):
```dart
const textFilter = "Paris";
final coords = Coordinates(latitude: 45, longitude: 10);
final preferences = SearchPreferences(
  maxMatches: 40,
  allowFuzzyResults: true,
  searchMapPOIs: true,
  searchAddresses: false,
);

final categories = GenericCategories.categories;
final firstCategory = categories[0];
final secondCategory = categories[1];

preferences.landmarks.addStoreCategoryId(
  firstCategory.landmarkStoreId,
  firstCategory.id,
);
preferences.landmarks.addStoreCategoryId(
  secondCategory.landmarkStoreId,
  secondCategory.id,
);

TaskHandler? taskHandler = SearchService.searchAroundPosition(
  coords,
  textFilter: textFilter,
  preferences: preferences,
  (err, results) async {
    if (err == GemError.success) {
      if (results.isEmpty) {
        showSnackbar("No results");
      } else {
        showSnackbar("Number of results: ${results.length}");
      }
    } else {
      showSnackbar("Error: $err");
    }
  },
);
```

Set the `searchAddresses` to false in order to filter non-relevant results.

### Search on custom landmarks

By default all search methods operate on the landmarks provided by default on the map. You can enable search functionality for custom landmarks by creating a landmark store containing the desired landmarks and adding it to the search preferences.
```dart
// Create the landmarks to be added
Landmark landmark1 = Landmark()
  ..coordinates = Coordinates(latitude: 25, longitude: 30)
  ..name = "My Custom Landmark1";

Landmark landmark2 = Landmark()
  ..coordinates = Coordinates(latitude: 25.005, longitude: 30.005)
  ..name = "My Custom Landmark2";

// Create a store and add the landmarks
LandmarkStore store = LandmarkStoreService.createLandmarkStore('LandmarksToBeSearched');
store.addLandmark(landmark1);
store.addLandmark(landmark2);

// Add the store to the search preferences
SearchPreferences preferences = SearchPreferences();
preferences.landmarks.add(store);

// If no results from the map POIs should be returned then searchMapPOIs should be set to false
preferences.searchMapPOIs = false;

// If no results from the addresses should be returned then searchAddresses should be set to false
preferences.searchAddresses = false;

// Search for landmarks
SearchService.search(
  "My Custom Landmark",
  Coordinates(latitude: 25.003, longitude: 30.003),
  preferences: preferences,
  (err, results) {
    if (err == GemError.success) {
      if (results.isEmpty) {
        showSnackbar("No results");
      } else {
        showSnackbar("Number of results: ${results.length}");
      }
    } else {
      showSnackbar("Error: $err");
    }
  },
);
```

The landmark store **retains** the landmarks added to it across sessions, until the app is **uninstalled**. This means a previously created landmark store with the same name might already exist in persistent storage and may contain pre-existing landmarks. For more details, refer to the [documentation on LandmarkStore](/guides/core/landmarks#landmark-stores).

Set the `searchAddresses` and `searchMapPOIs` to false in order to filter non-relevant results.

### Search on overlays

You can perform searches on overlays by specifying the overlay ID. It is recommended to consult the [Overlay documentation](/guides/core/overlays) for a deeper understanding and details about proper usage.

In the example below, we demonstrate how to search within items from the safety overlay. Custom overlays can also be used, provided they are activated in the applied map style:
```dart
// Get the overlay id of safety
int overlayId = CommonOverlayId.safety.id;

// Add the overlay to the search preferences
SearchPreferences preferences = SearchPreferences();
preferences.overlays.add(overlayId);

// We can set searchMapPOIs and searchAddresses to false if no results from the map POIs and addresses should be returned
preferences.searchMapPOIs = false;
preferences.searchAddresses = false;

TaskHandler? taskHandler = SearchService.search(
  "Speed",
  Coordinates(latitude: 48.76930, longitude:  2.34483),
  preferences: preferences,
  (err, results) {
    if (err == GemError.success) {
      if (results.isEmpty) {
        showSnackbar("No results");
      } else {
        mapController.centerOnCoordinates(results.first.coordinates);
        showSnackbar("Number of results: ${results.length}");
      }
    } else {
      showSnackbar("Error: $err");
    }
  },
);
```

In order to convert the returned `Landmark` to a `OverlayItem` use the `overlayItem` getter of the `Landmark` class.
This methods returns the associated `OverlayItem` if available, null otherwise.

Set the `searchAddresses` and `searchMapPOIs` to false in order to filter non-relevant results.

## Search for location

If you don't specify any text, all the landmarks in the closest proximity are returned, limited to `maxMatches`.
```dart
final coords = Coordinates(latitude: 45, longitude: 10);
final preferences = SearchPreferences(
  maxMatches: 40,
  allowFuzzyResults: true,
);

TaskHandler? taskHandler = SearchService.searchAroundPosition(
  coords,
  preferences: preferences,
  (err, results) async {
    // If there is an error or there aren't any results, the method will return an empty list.
    if (err == GemError.success) {
      if (results.isEmpty) {
        showSnackbar("No results");
      } else {
        showSnackbar("Number of results: ${results.length}");
      }
    } else {
      showSnackbar("Error: $err");
    }
  },
);
```

To limit the search to a specific area, provide a `RectangleGeographicArea` to the optional `locationHint` parameter.
```dart
final coords = Coordinates(latitude: 41.68905, longitude: -72.64296);
//highlight-start
final searchArea = RectangleGeographicArea(
    topLeft: Coordinates(latitude: 41.98846, longitude: -73.12412),
    bottomRight: Coordinates(latitude: 41.37716, longitude: -72.02342));
//highlight-end
SearchService.search('N', coords, (err, result) {
  successfulSearchCompleter.complete((err, result));
},
    preferences: SearchPreferences(maxMatches: 400),
    //highlight-start
    locationHint: searchArea,
    //highlight-end
    );
```

The reference coordinates used for search must be located within the `RectangleGeographicArea` provided to the `locationHint` parameter. Otherwise, the search will return an empty list.

## Show the results on the map

In most use cases the landmarks found by search are already present on the map.
If the search was made on custom landmark stores see the [add map landmarks](/guides/maps/display-map-items/display-landmarks#display-custom-landmarks) section for adding landmarks to the map.

To zoom to a landmark found via search, we can use ``GemMapController.centerOnCoordinates`` on the coordinates of the landmark found (``Landmark.coordinates``). See the documentation for [map centering](/guides/maps/adjust-map#map-centering) for more info.

## Change the language of the results

The language of search results and category names is determined by the `SdkSettings.language` setting. Check the [the internationalization guide](/guides/get-started/internationalization) section for more details.

## Relevant examples demonstrating search related features

- [Text Search](/examples/places-search/text-search)

- [Address Search](/examples/places-search/address-search)

- [Search Category](/examples/places-search/search-category)

- [Search Location](/examples/places-search/search-location)

- [What is Nearby](/examples/places-search/what-is-nearby)

- [Search Along Routes](/examples/places-search/search-along-route)

- [Location Wikipedia](/examples/places-search/location-wikipedia)
