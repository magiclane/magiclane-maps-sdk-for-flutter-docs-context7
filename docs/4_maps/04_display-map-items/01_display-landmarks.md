---
description: Documentation for Display Landmarks
title: Display Landmarks
---

# Display landmarks

# Filter landmarks by category

When displaying the map, we can choose what types of landmarks we want to display. Each landmark can have one or more `LandmarkCategory`. To selectively display specific categories of landmarks programmatically, you can use the `addStoreCategoryId` method provided by the `LandmarkStoreCollection` class:
```dart
// Clear all the landmark types on the map
mapController.preferences.lmks.clear();

// Display only gas stations
mapController.preferences.lmks.addStoreCategoryId(
GenericCategories.landmarkStoreId, GenericCategoriesId.gasStation.id);
```

This allows filtering the default map data. Next, we might want to also add custom landmarks to the map (see the next section).

## Display custom landmarks

Landmarks can be added to the map by storing them in a ``LandmarkStore``. Next, the ``LandmarkStore`` is added to the ``LandmarkStoreCollection`` within ``MapViewPreferences``. The following code demonstrates all these steps: first, creating custom landmarks, then adding them to a store, and finally adding the store to the collection of stores.
```dart
final List<Landmark> landmarksToAdd = [];
final Img imageData = await Img.fromAsset('assets/poi83.png');

final landmark1 = Landmark();

landmark1.name = "Added landmark1";
landmark1.coordinates = Coordinates(latitude: 51.509865, longitude: -0.118092);
landmark1.img = imageData;
landmarksToAdd.add(landmark1);

final landmark2 = Landmark();

landmark2.name = "Added landmark2";
landmark2.coordinates = Coordinates(latitude: 51.505165, longitude: -0.112992);
landmark2.img = imageData;
landmarksToAdd.add(landmark2);

final landmarkStore = LandmarkStoreService.createLandmarkStore('landmarks');

for (final lmk in landmarksToAdd) {
  landmarkStore.addLandmark(lmk);
}

mapController.preferences.lmks.add(landmarkStore);
```

Some methods of ``LandmarkStoreCollection`` class are explained below:

- ``add(LandmarkStore lms)``: add a new store to be displayed on map. All the landmarks from the store will be displayed, regardless of the category provided.

- ``addAllStoreCategories(int storeId)``: does the same thing as `add` but uses the `storeId`, not the `LandmarkStore` instance.

- ``addStoreCategoryId(int storeId, int categoryId)``: adds the landmarks with the specified category from the landmark store on the map.

- ``clear()``: removes all landmark stores from the map.

- ``contains(int storeId, int categoryId)``: check if the specified category ID from the specified store ID was already added.

- ``containsLandmarkStore(LandmarkStore lms)``: check if the specified store has any categories of landmarks shown on the map.

## Highlight landmarks

Highlights allow customizing a list of landmarks, making them more visible and providing options to customize the render settings. Highlights can only be used upon Landmarks. By default, highlighted landmarks are not selectable but can be made selectable if necessary.

Highlighting a landmark allows:

  - Customizing its appearance.

  - Temporarily isolating it from standard interactions - it cannot be selected (default behavior, can be modified for each highlight).

Landmarks retrieved through search or other means can be highlighted to enhance their prominence and customize their appearance. Additionally, custom landmarks can be highlighted, but they have to be added to a `LandmarkStore` first.

Highlights can be displayed on map by using ``GemMapController.activeHighlight``. For demonstration purposes, a new ``Landmark`` object will be instantiated and filled with specifications using available setters. Next, the created landmark needs to be added to a ``List<Landmarks>`` and a ``HighlightRenderSettings`` needs to be provided in order to enable desired customizations. Then ``activateHighlight`` can be called with an unique ``highlightId``.
```dart
final List<Landmark> landmarksToHighlight = [];
final Img imageData = await Img.fromAsset('assets/poi83.png');

final landmark = Landmark();

landmark.name = "New Landmark";
landmark.coordinates = Coordinates(latitude: 52.48209, longitude: -2.48888);
landmark.img = imageData;
landmark.extraImg = imageData;
landmarksToHighlight.add(landmark);

final settings = HighlightRenderSettings(imgSz: 50, textSz: 10, options: {
  HighlightOptions.noFading,
  HighlightOptions.overlap,
});

    
final lmkStore = LandmarkStoreService.createLandmarkStore('landmarks');
lmkStore.addLandmark(landmark);

controller.preferences.lmks.add(lmkStore);

mapController.activateHighlight(
  landmarksToHighlight,
  renderSettings: settings,
  highlightId: 2,
);

mapController.centerOnCoordinates(Coordinates(latitude: 52.48209, longitude: -2.48888), zoomLevel: 40);
```

To remove a displayed landmark from the map, use ``GemMapController.deactivateHighlight(id)`` to selectively remove a specific landmark, or ``GemMapController.deactivateAllHighlights()`` to clear all displayed landmarks at once.
```dart
mapController.deactivateHighlight(highlightId: 2);
```

To get the highlighted landmarks based on a particular highlight id:
```dart
List<Landmark> landmarks = mapController.getHighlight(2);
```

Overlay items can also be highlighted using the `activateHighlightOverlayItems` method in a similar way.

## Disabling landmarks

Removing all presented landmarks can be done by calling `removeAllStoreCategories(GenericCategories.landmarkStoreId)` method of `LandmarkStoreCollection` class. The following code does just that:
```dart
mapController.preferences.lmks
    .removeAllStoreCategories(GenericCategories.landmarkStoreId);
```


