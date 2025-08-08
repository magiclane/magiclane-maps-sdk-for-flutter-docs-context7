---
description: Documentation for Migrate To 2 16 0
title: Migrate To 2 16 0
---

# Migrate to 2.16.0

This guide outlines the breaking changes introduced in SDK version 2.16.0. Required updates may vary depending on your use case.

Additionally, new features and bugfixes have been introduced and are **not** documented here. For a comprehensive list of changes, please refer to the changelog.

This release brings small improvements to the API and API reference, better error handling and bugfixes.

## Removed *DataBuffer* and dependent methods

The `DataBuffer` class has been removed as it was no longer relevant for the Flutter SDK and it was not working. The same functionality can be achieved through the `Uint8List` predefined type.

As a result, the following methods that used `DataBuffer` were also removed:

- `updateCurrentStyleFromJson` method from the `MapViewPreferences` class

- `save`, `load` methods from the `MapViewMarkerCollections` class

## The return type related to *DataSource* instantiation now return *DataSource?* instead of *DataSource*

The affected methods are `createLogDataSource`, `createLiveDataSource`, `createExternalDataSource` and `createSimulationDataSource` static methods from the `DataSource` class.
They now return `DataSource?` instead of `DataSource`. The value null is returned when the instantiation failed. These methods no longer throw the `GemError` code if the instantiation failed.
The error code can now be obtained via the `ApiErrorService.apiError` getter.

Before:
```dart
try {
    DataSource dataSource = DataSource.createLiveDataSource();
    // Do something with the dataSource
} on GemError catch (error) {
    print("The data source instance creation failed: $error");
}
```

After:
```dart
DataSource? dataSource = DataSource.createLiveDataSource();
if (dataSource != null){
    // Do something with the dataSource
} else {
    print("The data source instance creation failed: ${ApiErrorService.apiError}");
}
```

This change improves the consistency of the API and better emphasizes the possibility of operation failure.

## Parameters renamed for the *setAllowConnection*  method of the *SdkSettings* class

The `allow` parameter has been renamed to `allowInternetConnection` and the `canDoAutoUpdate` parameter has been renamed to `canDoAutoUpdateResources`.
This change better reflects what the parameters are controlling.

Before:
```dart
SdkSettings.setAllowConnection(true, canDoAutoUpdate: true, ...);
```

After:
```dart
SdkSettings.setAllowConnection(true, canDoAutoUpdateResources: true, ...);
```

## The *hitTest* method from the *MapViewMarkerCollections* class now returns list of *MarkerMatch* instead of *MarkerMatchList*

Before:
```dart
RectangleGeographicArea area = ...
MapViewMarkerCollections collection = ...
MarkerMatchList matchList = collection.hitTest(area);
List<MarkerMatch> result = matchList.toList();
```

After:
```dart
RectangleGeographicArea area = ...
MapViewMarkerCollections collection = ...
List<MarkerMatch> result = collection.hitTest(area);
```

## The *MarkerInfoSpecialAccess* class is no longer public.

The class is for internal use only and should not have been exposed in the API.
Please use the relevant methods from the `MapViewMarkerCollections` and `MarkerCollection` classes instead.
