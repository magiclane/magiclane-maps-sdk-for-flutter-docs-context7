---
description: Documentation for Display Markers
title: Display Markers
---

# Display markers

The base class for the marker hierarchy is `Marker`. It encapsulates coordinates assigned to a specific part. Multiple coordinates can be added to the same marker and be separated into different parts. If no part is specified, the coordinates are added to a default part, indexed as 0. The coordinates are stored in a list-like structure, where you can specify their index explicitly. By default, the index is set to -1, meaning the coordinate will be appended to the end of the list.
```dart
// code used for displaying a marker with coordinates separated into different parts
final marker1 = Marker()
  ..add(Coordinates(latitude: 52.1459, longitude: 1.0613), part: 0)
  ..add(Coordinates(latitude: 52.14569, longitude: 1.0615), part: 0)
  ..add(Coordinates(latitude: 52.14585, longitude: 1.06186), part: 1)
  ..add(Coordinates(latitude: 52.14611, longitude: 1.06215), part: 1);
```


```dart
// code used for displaying a marker with coordinates added to the same part
final marker1 = Marker()
  ..add(Coordinates(latitude: 52.1459, longitude: 1.0613), part: 0)
  ..add(Coordinates(latitude: 52.14569, longitude: 1.0615), part: 0)
  ..add(Coordinates(latitude: 52.14585, longitude: 1.06186), part: 0)
  ..add(Coordinates(latitude: 52.14611, longitude: 1.06215), part: 0);
```

To display any type of marker on a map, it must first be added to a `MarkerCollection`. Creating a collection of markers requires providing a name and specifying the desired `MarkerType` enum as parameters for its constructor. The collection of markers displayed above used `MarkerType.polyline`, but it can also be `MarkerType.point` or `MarkerType.polygon`.

Once the `MarkerCollection` object has been populated, it must be added to the `MapViewMarkerCollections` field within the `MapViewPreferences` class. This can be accessed through the `GemMapController`, as shown below:
```dart
mapController.preferences.markers.add(markerCollection);
```

### Point Type Marker

Visually represented as an icon, it is used to dynamically highlight user-defined locations. To display a point-type marker, the `MarkerCollection` to which the markers are added must be of the `MarkerType.point` type.
```dart
final marker = Marker()
  ..add(Coordinates(latitude: 52.1459, longitude: 1.0613), part: 0)
  ..add(Coordinates(latitude: 52.14569, longitude: 1.0615), part: 0)
  ..add(Coordinates(latitude: 52.14585, longitude: 1.06186), part: 1)
  ..add(Coordinates(latitude: 52.14611, longitude: 1.06215), part: 1);

final markerCollection = MarkerCollection(markerType: MarkerType.point, name: "myCollection");

markerCollection.add(marker);

mapController.preferences.markers.add(markerCollection);
```

The result will be the following:

By default, point-type markers appear as blue circles up to a specific zoom level. When the zoom threshold is exceeded, they automatically cluster into orange circles, and at higher levels of clustering, they transition to red circles. Learn more at [Marker Clustering](#marker-clustering)

### Polyline Type Marker

This type of marker is designed to display a continuous line consisting of one or more connected straight-line segments. To use it, ensure the `MarkerRenderSettings` specifies `markerType` as `MarkerType.polyline`. It's important to note that markers can include multiple coordinates, which may or may not belong to the same part. Coordinates within the same part are connected by a polyline, which is red by default, while coordinates outside the part remain unconnected.

For more information, see [Markers section](/guides/maps/display-map-items/display-markers).

### Polygon Type Marker

This type of marker is designed to display a closed two-dimensional figure composed of straight-line segments that meet at their endpoints. To use it, ensure the `MarkerRenderSettings` specifies `markerType` as `MarkerType.polygon`.

To successfully create a polygon, at least three coordinates must be added to the same part. Otherwise, the result will be an open polyline rather than a closed shape.

Polygons can be customized using properties like `polygonFillColor` and `polygonTexture`. Additionally, since polygon edges are essentially polylines, you can further refine their appearance with polyline-related attributes such as `polylineInnerColor`, `polylineOuterColor`, `polylineTexture`, and more.

### Marker Customizations

To customize the appearance of markers on GemMap, you can use the `MarkerCollectionRenderSettings` class.

This class is designed for customizing the appearance of individual markers. It includes various fields that can influence a marker's appearance, regardless of its type, as it provides customizable features for all marker types. For example:

- For markers of type `MarkerType.polyline`, you can use fields such as `polylineInnerColor`, `polylineOuterColor`, `polylineInnerSize`, and `polylineOuterSize`.

- For `MarkerType.polygon`, the `polygonFillColor`, `polygonTexture` fields are available, among others.

- For `MarkerType.point`, you can use fields such as `labelTextColor`, `labelTextSize`, `image`, `imageSize`.

All dimensional sizes (`imageSize`, `textSize`, etc.) are measured in millimeters.

If customizations unrelated to a marker's specific type are applied—for example, using `polylineInnerColor` for a `MarkerType.point`—they will simply be ignored, and the marker's appearance will remain unaffected.

For `MarkerType.point`, a key customizable field is `labelingMode`. This field is a set that consists of values from `MarkerLabelingMode` enum. This allows you to enable desired features, such as positioning the label text above the icon or placing the icon above the marker's coordinates, by adding them to the `labelingMode` set as shown below:
```dart
final renderSettings = MarkerCollectionRenderSettings(labelingMode: {
      MarkerLabelingMode.itemLabelVisible,
      MarkerLabelingMode.textAbove,
      MarkerLabelingMode.iconBottomCenter
    });

mapController.preferences.markers.add(markerCollection, settings: renderSettings);
```

To hide a marker's name or its group's name, create a `MarkerCollectionRenderSettings` object with a `labelingMode` that excludes `MarkerLabelingMode.itemLabelVisible` and `MarkerLabelingMode.groupLabelVisible`. By default, both options are enabled.

The above code will result in the following marker appearance:

To assign a name to a marker, use the name setter of the `Marker` class.

To customize the icons of the displayed markers, add the collection to `MapViewMarkerCollections` and configure a `MarkerCollectionRenderSettings` instance with the relevant image field. This field controls the appearance of the entire collection.
```dart
import 'package:flutter/services.dart' show rootBundle;

final ByteData imageData = await rootBundle.load('assets/poi83.png');
final Uint8List pngImage = imageData.buffer.asUint8List();

final renderSettings = MarkerCollectionRenderSettings(image: GemImage(image: pngImage, format: ImageFileFormat.png));
```

Code above is setting a custom icon to a marker. The result is the following:

#### Marker Sketches

To customize the appearance of each marker individually, use the `MarkerSketches` class, which extends `MarkerCollection`. This lets you define unique styles and properties for every marker. You can obtain a MarkerSketches object using the `MapViewMarkerCollections.getSketches()` method:
```dart
final sketches = ctrl.preferences.markers.getSketches(MarkerType.point);
```

Typical operations are adding a sketch with an optional per‑marker render configuration and position, reading a sketch’s rendering configuration.

:::tip[note]

There are only three `MarkerSketches` collections, one for each marker type: `MarkerType.point`, `MarkerType.polyline`, and `MarkerType.polygon`. Each collection is singleton.

Adding markers to a `MarkerSketches` collection is similar to adding them to a `MarkerCollection`. However, when adding markers to a `MarkerSketches` collection, you can specify individual `MarkerRenderSettings` and index for each marker. This allows for greater customization of each marker's appearance.
```dart
final marker1 = Marker()..add(Coordinates(latitude: 39.76741, longitude: -46.8962));
marker1.name = "HelloMarker";

sketches.addMarker(marker1,
    settings: MarkerRenderSettings(
    labelTextColor: Colors.red,
    labelTextSize: 3.0,
    image: GemImage(imageId: GemIcon.toll.id)),
      index: 0);
```

In order to change a marker's appearance after it has been added to a `MarkerSketches` collection, you can use `setRenderSettings` method:
```dart
sketches.setRenderSettings(
    0, // marker index
    MarkerRenderSettings(
      labelTextColor: Colors.red,
      labelTextSize: 3.0,
      image: GemImage(imageId: GemIcon.toll.id))
);
```

In order to obtain the current render settings of a marker, you can use `getRenderSettings` method called with the marker index:
```dart
final returnedSettings = sketches.getRenderSettings(0);
```

Calling `getRenderSettings` with an invalid index will return a `MarkerRenderSettings` object with default values.

The `MarkerSketches` collection does not need to be added to `MapViewMarkerCollections`, as it is already part of it. Any changes made to the `MarkerSketches` collection will be automatically reflected on the map.

Adding a `MarkerSketches` object to `MapViewMarkerCollections` with `MarkerCollectionRenderSettings` will be overwritten by the individual `MarkerRenderSettings` of markers from the collection. 

### Marker Clustering

Clustering or grouping is a default feature of markers. Beyond a certain zoom level, the markers automatically cluster into a single marker containing a number of items lesser than `lowGCountDefault` if the group is a low density one. The image of those groups can be customized with `lowDensityPointsGroupImage`, `mediumDensityPointsGroupImage`, `highDensityPointsGroupImage` fields of `MarkerCollectionRenderSettings`. The number of markers contained by a group can be set through `lowDensityPointsGroupMaxCount`, `mediumDensityPointsGroupMaxCount`.
```dart
// code for markers not grouping at zoom level 70
final renderSettings = MarkerCollectionRenderSettings();

mapController.preferences.markers.add(markerCollection, settings: renderSettings);

mapController.centerOnCoordinates(Coordinates(latitude: 52.14611, longitude: 1.06215), zoomLevel: 70);
```


```dart
// code for markers grouping at zoom level 70
final renderSettings = MarkerCollectionRenderSettings(labelTextSize: 3.0, labelingMode: labelingMode, pointsGroupingZoomLevel: 70);

mapController.preferences.markers.add(markerCollection, settings: renderSettings);

mapController.centerOnCoordinates(Coordinates(latitude: 52.14611, longitude: 1.06215), zoomLevel: 70);
```

You can disable marker clustering by setting the `pointGroupingZoomLevel` to 0. However, note that doing so for a large number of markers may significantly impact performance, as rendering each individual marker increases GPU resource usage.

Marker clusters are represented by the first marker from the collection as the **group head**. The group head marker is returned by the `getPointsGroupHead` method:
```dart
final markerCollection = MarkerCollection(
    markerType: MarkerType.point, name: "Collection1");

final marker1 = Marker()
  ..add(Coordinates(latitude: 39.76717, longitude: -46.89583));
marker1.name = "NiceName";
final marker2 = Marker()
  ..add(Coordinates(latitude: 39.767138, longitude: -46.895640));
marker2.name = "NiceName2";
final marker3 = Marker()
  ..add(Coordinates(latitude: 39.767145, longitude: -46.895690));
marker3.name = "NiceName3";

markerCollection.add(marker1);
markerCollection.add(marker2);
markerCollection.add(marker3);

ctrl.preferences.markers.add(markerCollection,
//highlight-start
    settings:
        MarkerCollectionRenderSettings(buildPointsGroupConfig: true));
//highlight-end

// This centering triggers marker grouping
ctrl.centerOnCoordinates(
    Coordinates(latitude: 39.76717, longitude: -46.89583),
    zoomLevel: 50);

// Wait for the center process to finish
await Future.delayed(Duration(milliseconds: 250));

//highlight-start
final marker = markerCollection.getPointsGroupHead(marker2.id); // Returns marker1
//highlight-end
```

Since marker grouping depends on the loading of tiles at a certain zoom level, you need to wait for them to load; otherwise, calling `getPointsGroupHead` will return a reference to the queried marker, because the markers are not yet grouped. Thus `getPointsGroupComponents` will return an empty list.

This behavior occurs only when the `MarkerCollection` is added to `MapViewMarkerCollections` using **MarkerCollectionRenderSettings(buildPointsGroupConfig: true)** and the markers are **grouped** based on the zoom level. In all other cases, the method returns a direct reference to the queried marker.

All markers from a group can be returned by using `getPointsGroupComponents` method called with the head marker id, returned by `MarkerCollection.getPointsGroupHead` method, which is considered the `groupId`. This method returns all markers except the group head marker.
```dart
final marker = markerCollection.getPointsGroupHead(marker2.id);

//highlight-start
final groupMarkers =
  markerCollection.getPointsGroupComponents(marker.id);
//highlight-end
```

If `getPointsGroupComponents` is not invoked with the ID of the group head marker, the method will return an empty list.

### Adding large amount of markers

If there is a need for adding lots of markers at the same time, this can be done much more efficiently through `addList` method of `MapViewMarkerCollection`. This uses a list of `MarkerWithRenderSettings` objects, which consists of an `MarkerJson` (essentially, it’s a marker consisting of a list of coordinates and an associated marker name) and an `MarkerRenderSettings`. The following example showcases how it works:
```dart
List<MarkerWithRenderSettings> markers = [];

for (int i = 0; i < 8000; ++i) {
  // Generate random coordinates to display some markers.
  double randomLat = minLat + random.nextDouble() * (maxLat - minLat);
  double randomLon = minLon + random.nextDouble() * (maxLon - minLon);

  final marker = MarkerJson(
    coords: [Coordinates(latitude: randomLat, longitude: randomLon)],
    name: "POI $i",
  );

  // Choose a random POI icon for the marker and set the label size.
  final renderSettings = MarkerRenderSettings(
      image: GemImage(
          image: listPngs[random.nextInt(listPngs.length)],
          format: ImageFileFormat.png),
      labelTextSize: 2.0);

  // Create a MarkerWithRenderSettings object.
  final markerWithRenderSettings =
      MarkerWithRenderSettings(marker, renderSettings);

  // Add the marker to the list of markers.
  markers.add(markerWithRenderSettings);
}

// Create the settings for the collections.
final settings = MarkerCollectionRenderSettings();

// Set the label size.
settings.labelGroupTextSize = 2;

// The zoom level at which the markers will be grouped together.
settings.pointsGroupingZoomLevel = 35;

// Set the image of the collection.
settings.image = GemImage(image: imageBytes, format: ImageFileFormat.png);
// To delete the list you can use this method: mapController.preferences.markers.clear();

// Add the markers and the settings on the map.
mapController.preferences.markers.addList(list: markers, settings: settings, name: "Markers");
```


