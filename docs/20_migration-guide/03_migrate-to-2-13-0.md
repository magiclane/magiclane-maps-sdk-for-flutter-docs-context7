---
description: Documentation for Migrate To 2 13 0
title: Migrate To 2 13 0
---

# Migrate to 2.13.0

This guide outlines the breaking changes introduced in SDK version 2.13.0. Required updates may vary depending on your use case.

Additionally, new features and bugfixes have been introduced and are **not** documented here. For a comprehensive list of changes, please refer to the changelog.

This release provides more sense additions and brings fixes to many issues.

## Most images are now nullable. The return value changed from *Uint8List* to *Uint8List?*

Most image related methods from the SDK are impacted by this change.

Before:
```dart
NavigationInstruction instruction = ...
Uint8List image = instruction.getLaneImage(size: const Size(100, 100), format: ImageFileFormat.png);
// Use the image...
```

After:
```dart
NavigationInstruction instruction = ...
Uint8List? image = instruction.getLaneImage(size: const Size(100, 100), format: ImageFileFormat.png);
if (image != null){
    // Use the image...
} else {
    print('No valid lane image for current instruction');
}
```

Methods might return null result if the image was invalid or not available. Before this change there was no reliable way to check if a returned image is valid.
Affected methods include `TurnDetails.getAbstractGeometryImage`, `SignpostDetails.getImage`, `SdkSettings.getImageById`, `NavigationInstruction.getLaneImage`, `NavigationInstruction.getNextNextTurnImage`, `NavigationInstruction.getNextTurnImage`, `MapDetails.getCountryFlag`, `LandmarkCategory.getImage`, `Landmark.getExtraImage`, `Landmark.getImage`, `ContentStoreItem.getImagePreview`, `RouteTrafficEvent.getImage`.

## Flutter version 3.27 might be required

The API users might be required to update their code for the new Flutter version. This requirement is the result of an internal change related to the [wide gamut color change](https://docs.flutter.dev/release/breaking-changes/wide-gamut-framework) added in the Flutter 3.27 release.

If compile errors related to the `Color` dart class are encountered, please update Flutter to the latest version:
```bash
$ flutter upgrade
```

## Renamed register methods from the *GemMapController* class

Renamed `registerOnLongPressCallback`, `registerOnMapAngleUpdateCallback` and `registerOnMapViewMoveStateChangedCallback` methods from the `GemMapController` class to `registerLongPressCallback`, `registerMapAngleUpdateCallback`, `registerMapViewMoveStateChangedCallback`. Old methods are still available but they are deprecated and will be removed in a future release.

Before:
```dart
GemMapController controller = ...
controller.registerOnMapAngleUpdateCallback((angle){...});
```

After:
```dart
GemMapController controller = ...
controller.registerMapAngleUpdateCallback((angle){...});
```

This change brings consistency to the methods names. The old methods are deprecated and will be removed in a future release.

## The *create* static method from the *RecorderBookmarks* class returns *RecorderBookmarks?* instead of *RecorderBookmarks*

Before:
```dart
RecorderBookmarks bookmarks = RecorderBookmarks.create(trackDir)
// Use bookmarks...
```

After:
```dart
RecorderBookmarks? bookmarks = RecorderBookmarks.create(trackDir)
if (bookmarks != null) {
    // Use bookmarks...
} else {
    print('RecorderBookmarks creation failed. Invalid path passed.')
}
```

The value `null` will be returned if the `RecorderBookmarks` object couldn't be created based on the provided path.

## The *stopRecording* method from the *Recorder* is now async

Not awaiting the `stopRecording` method might lead into unexpected and unpredictable behaviour.

Before:
```dart
Recorder recorder = ...

GemError error = await recorder.startRecording();
// Do operations with the recorder...
GemError stopError = await recorder.stopRecording();
```

After:
```dart
Recorder recorder = ...

GemError error = await recorder.startRecording();
// Do operations with the recorder...
GemError stopError = await recorder.stopRecording();
```

## The *getOverlayById* and *getOverlayAt* methods from the *OverlayCollection* class return *OverlayItem?* instead of *OverlayItem*

Before:
```dart
OverlayCollection collection = ...;
OverlayInfo item = collection.getOverlayById(1);
// Do operations with the item...
```

After:
```dart
OverlayCollection collection = ...;
OverlayInfo? item = collection.getOverlayById(1);
if (item != null){
    // Do operations with the item...
} else {
    print('No overlay item with the given id was found');
}
```

The value `null` will be returned if the input provided to the method is not valid.

## The *alignNorthUp* method of the *MapView* class takes a parameter of type *GemAnimation* instead of *Duration*

Before:
```dart
GemMapController controller = ...;
controller.alignNorthUp(duration: Duration(seconds: 1));
```

After:
```dart
GemMapController controller = ...;
GemAnimation animation = GemAnimation(duration: 1000, type: AnimationType.linear);
controller.alignNorthUp(animation: animation);
```

This change allows more configuration and flexibility for the `alignNorthUp` method.

## Most listeners no longer implement *EventDrivenProgressListener*. They now implement *EventHandler*.

SDK users who relied on polymorphic behavior of the listeners in their code may need to adjust their implementation to accommodate these changes.

Before:
```dart
ProgressListener listener = RouteListener();
```

Now:
```dart
EventHandler listener = RouteListener();
```

The `registerOnCompleteWithDataCallback`, `registerOnProgressCallback`, `registerOnNotifyStatusChanged` methods and `progressMultiplier`, `notifyProgressInterval` getters and `notifyProgressInterval` setter of the `ProgressListener` interface are no longer available in the affected classes. More internal members are also no longer available for the affected classes.

The affected classes are `GemView`, `NetworkProvider`, `RouteListener`, `OffboardListener`, `NavigationListener`, `LandmarkStoreListener`, `AlarmListener`, `PositionListener`, `DataSourceListener`.

As a consequence of this change, the `registerOnNotifyCustom` method was no longer required and was removed.

## *getPosition* and *getImprovedPosition* methods of the *PositionService* changed to *position* and *improvedPosition* getters

Before:
```dart
Position position = PositionService.instance.getPosition();
```

After:
```dart
Position position = PositionService.instance.position;
```

The old methods are now deprecated and will be removed in a future release.

## The *getCountryLevelItem* method of the *GuidedAddressSearchService* returns *Landmark?* instead of *Landmark*

Before:
```dart
Landmark parentLmk = GuidedAddressSearchService.getCountryLevelItem('INVALID INPUT');
// Do something with the parentLmk.
```

After:
```dart
Landmark? parentLmk = GuidedAddressSearchService.getCountryLevelItem('INVALID INPUT');
if (parentLmk != null){
    // Do something with the parentLmk.
} else {
    print('The country with the given code was not found.');
}
```

The value `null` is returned if no country with the given code exists.

## The *exportAs* methods of the *Route* and *Path* classes return *String* instead of *Uint8List*

Before:
```dart
Route route = ...;
Uint8List binaryGpx = route.exportAs(PathFileFormat.gpx);
String gpxString = utf8.decode(binaryGpx);
Uint8List decodedGpxData = base64.decode(gpxString);

// The decodedGpxData can be written to a file using the writeAsBytes method
```

After:
```dart
Route route = ...;
String stringGpx = route.exportAs(PathFileFormat.gpx);

// The decodedGpxData can be written to a file using the writeAsString method
// or it can be used directly in your application. It is in a human readable format.
```

## The *getMapExtendedCapabilities* method from the *MapDetails* class returns *MapExtendedCapability* set instead of *int*

Before:
```dart
int capabilities = MapDetails.getMapExtendedCapabilities();
```

After:
```dart
Set<MapExtendedCapability> capabilities = MapDetails.getMapExtendedCapabilities();
```

## The *serializeListOfMarkers* method has been removed from the public API

The `serializeListOfMarkers` is no longer accessible through the public API. Please use the `addList` method of the `MapViewMarkerCollections` class instead.

## DataType enum value changes

The `heartRate` value was added to the `DataType` enum. The wrongly named `altitude` value was replaced with `attitude`.
