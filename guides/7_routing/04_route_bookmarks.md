---
description: Documentation for Route Bookmarks
title: Route Bookmarks
---

# Route Bookmarks

The `RouteBookmarks` class provides a way to store, manage, and retrieve collections of routes as bookmarks between application sessions.
This is useful for applications that need to save user trips, import/export routes, or manage multiple planned journeys.

## Creating a RouteBookmarks collection

To create a new bookmarks collection, use the static `RouteBookmarks.create` method and provide a unique name:
```dart
final bookmarks = RouteBookmarks.create('my_trips');
```

If a collection with the same name already exists, it will be opened instead.

The file path of the bookmarks collection can be accessed using the `filePath` property:
```dart
String path = bookmarks.filePath;
```

## Adding Routes

Add a new route to the collection using the `add` method. You must provide a unique route name and a list of waypoints. Optionally, you can include route preferences and specify whether to overwrite an existing route with the same name.
```dart
bookmarks.add(
  'Home to Office',
  [homeLandmark, officeLandmark],
  preferences: myPreferences,
  overwrite: false,
);
```

The `add` method takes the following parameters:

- `name` (`String`): The unique name for the route.

- `waypoints` (`List<Landmark>`): The list of landmarks defining the route.

- `preferences` (`RoutePreferences?`): Optional route preferences.

- `overwrite` (`bool`): If `true`, if there is already a route with the same name it will be replaced. The default value is `false`.

If a route with the same name exists and `overwrite` is `false`, the operation fails.

## Importing Routes

You can import multiple routes from a file using `addTrips`. The method returns the number of imported routes or `GemError.invalidInput.code` if the import fails.
```dart
final int count = bookmarks.addTrips('/path/to/bookmarks_file');
if (count == GemError.invalidInput.code){
    showSnackbar('Invalid file path provided for import.');
} else {
    showSnackbar('$count trips imported successfully.');
}
```

## Exporting a Route

Export a specific route to a file using `exportToFile`. Provide the route index and the destination file path.
```dart
final result = bookmarks.exportToFile(0, '/path/to/exported_route');
showSnackbar('Export completed with result: $result');
```

This method returns:

- `GemError.success` on success.

- `GemError.notFound` if the route does not exist.

- `GemError.io` if the file cannot be created.

## Accessing Routes

To get the number of routes in the collection, use the `size` property:
```dart
final int count = bookmarks.size;
```

To get details of a specific route by its index, use the following methods:
```dart
String? name = bookmarks.getName(0);
List<Landmark>? waypoints = bookmarks.getWaypoints(0);
RoutePreferences? prefs = bookmarks.getPreferences(0);
DateTime? timestamp = bookmarks.getTimestamp(0);
```

The methods return `null` if the index is out of bounds or if the requested data is unavailable.
The `getTimestamp` method returns the date and time when the route was added/modified.

To find the index of a route by its name, use the `find` method:
```dart
final index = bookmarks.find('Home to Office');
```

The `find` method returns:

- The route index if found (positive value).

- GemError.internalAbort.code if not found.

- GemError.notFound.code if the route is not found.

You can change the sort order of the bookmarks using the `sortOrder` property:
```dart
final int index = bookmarks.find('Home to Office');
if (index > 0) {
    showSnackbar('Route found at index $index.');
} else {
    showSnackbar('Error finding route: $index');
}
```

The available sort orders are:

  - `RouteBookmarksSortOrder.sortByDate` (default): Most recent first.

  - `RouteBookmarksSortOrder.sortByName`: Alphabetical order.

Enable or disable auto-delete mode using the `autoDeleteMode` property.

When enabled, the bookmarks database is deleted when the object is destroyed.

## Updating Routes

In order to update an existing route, use the `update` method with the route index and new details:
```dart
bookmarks.update(
    0,
    name: 'New Name',
    waypoints: [newStart, newEnd],
    preferences: newPrefs,
);
```

The `update` method only modifies the provided fields, leaving others unchanged.

## Removing Routes

To remove a route by its index, use the `remove` method:
```dart
bookmarks.remove(0);
```

To clear all routes from the collection, use the `clear` method:
```dart
bookmarks.clear();
```


