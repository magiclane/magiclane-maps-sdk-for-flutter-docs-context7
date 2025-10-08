---
description: Documentation for Manage Content
title: Manage Content
---

# Manage Content

The Maps SDK for Flutter offers comprehensive functionality for managing offline content.

The supported downloadable content types are defined in the `ContentType` enum:

- `viewStyleHighRes`: High-dpi screen optimized map styles that can be applied offline. These include both a selection of default styles and user-created styles from the studio, based on the API key.

- `viewStyleLowRes`: Low-dpi screen map styles, optimized for smaller file sizes while maintaining essential details.

- `roadMap` : Offline maps covering countries and regions. Within a downloaded region, users can perform searches, calculate routes, and navigate without an internet connection.

- `humanVoice`: Pre-recorded voice files used to deliver spoken navigation instructions and warnings.

For most use cases, the high-resolution map styles option is recommended over its low-resolution counterpart.

At this time, other values within the `ContentType` enum are not fully supported by the Flutter SDK.

The ContentStore class is responsible for managing and providing a list of downloadable items.
Each item is represented by the `ContentStoreItem` class, which encapsulates details such as name, image, type, version, and size.
Additionally, it offers operations for downloading and deleting content.

Ensure that the API token is both set and valid. Some operations might return `GemError.busy` if no valid API key is set.

Modifying downloaded maps (download, delete, update) may interrupt ongoing operations such as search, route calculation, or navigation. If this occurs, the `onComplete` callback will be triggered with a `GemError.invalidated` value.

## List Online Content

To retrieve a list of available content from the Magic Lane servers, use the `asyncGetStoreContentList` method from the `ContentStore` class.
This method returns a `ProgressListener?`, which allows the operation to be stopped if needed. If the operation fails to start, it returns `null`.

The method accepts the content type as an argument and a callback that provides:

- The operation error code

- A list of `ContentStoreItem` objects

- A flag indicating whether the list is cached
```dart
final ProgressListener? listener = ContentStore.asyncGetStoreContentList(ContentType.roadMap, (err, items, isCached){
    if (err != GemError.success){
    showSnackbar("Failed to get list of content store items: $err");
    } else {
        /// Do something with the items and isCached flag.
    }
});
```

The `asyncGetStoreContentList` method should be called only when an active internet connection is available and the current offline version is not expired. If no internet connection is available or if the current offline map version is expired, use the `getLocalContentList` method to retrieve the offline content list instead.

Do not invoke asyncGetStoreContentList within `onMapCreated` callback, as it will fail with `GemError.notFound` or `GemError.noConnection` (unless the request targets `ContentType.humanVoice`). Wait until map tiles are fully loaded before calling it.

## List Local Content

The `getLocalContentList` method can be used to get the list of local content available offline.
```dart
final List<ContentStoreItem> items = ContentStore.getLocalContentList(ContentType.roadMap);
/// Do something with the items
```

The `getLocalContentList` method returns content store items that are either ready for use or currently downloading, as well as those pending download.

## Filter Content 

To obtain a **filtered** list of available content from the Magic Lane servers, use the `asyncGetStoreFilteredList` method from the `ContentStore` class. You can filter content by specifying country ISO 3166-3 codes and by geographic area using a `RectangleGeographicArea`.
```dart
final contentStoreItemListCompleter = Completer<List<ContentStoreItem>?>();

ContentStore.asyncGetStoreFilteredList(
    type: contentType,
    area: RectangleGeographicArea(
        topLeft: Coordinates(latitude: 53.7731, longitude: -1.7990),
        bottomRight: Coordinates(latitude: 38.4549, longitude: 21.1696)),
    onComplete: (err, result) {
    contentStoreItemListCompleter.complete(result);
    });

  final res = await contentStoreItemListCompleter.future;
```

The `getStoreFilteredList` method returns the filtered content store items that were last requested via `asyncGetStoreFilteredList` method.

### Method behaviour

| Condition                                                                                         | `onComplete` Result                                                  |
|---------------------------------------------------------------------------------------------------|----------------------------------------------------------------------|
| The `countries` list contains invalid ISO 3166-3 codes                                            | `GemError.success` with an empty `ContentStoreItem` list.            |
| The `countries` list includes countries incompatible with the specified `RectangleGeographicArea` | `GemError.success` with an empty `ContentStoreItem` list.            |
| Insufficient memory to complete the operation                                                     | `GemError.noMemory` with an empty `ContentStoreItem` list.           |
| Invalid `GeographicArea` (e.g., invalid coordinates)                                              | `GemError.success` with a full list of `ContentStoreItem`; behaves as if no filter was applied. |
| The `area` parameter is an empty `TilesCollectionGeographicArea`                                  | `GemError.invalidInput` with an empty `ContentStoreItem` list.       |
| HTTP request failed                                                                               | `GemError.general` with an empty `ContentStoreItem` list.            |

## Content Store Item Fields

#### Fields Containing General Information

| Field Name         | Description                                                  |
|--------------------|--------------------------------------------------------------|
| `name`            | The name of the associated product, automatically translated. |
| `id`              | The unique ID of the item in the content store.               |
| `type`            | The type of the product as a `ContentType` value.             |
| `fileName`        | The full path to the content data file.                       |
| `totalSize`       | The total size of the content in bytes.                       |
| `availableSize`   | The downloaded size of the content.                           |
| `updateSize`      | The update size if an update is available.                    |
| `status`          | The current status of the item as a `ContentStoreItemStatus`. |
| `contentParameters`| Additional information about an item is the form of a `SearchableParameterList` object |
| `imgPreview`      | The image associated with the content store item. The user is responsible to check if the image is valid |

For checking if a `ContentStoreItem` is downloaded/available/downloading/updating use the `status` field value:

  - `unavailable`: The content store item is not downloaded and cannot be used.

  - `completed`: The content store item has been downloaded and is ready to be used.

  - `paused`: The download operation has been paused by the user.

  - `downloadQueued`: The download is queued and will proceed once resources are available.

  - `downloadWaitingNetwork`: No internet connection is established, and the download will proceed once a network connection is available.

  - `downloadWaitingFreeNetwork`: The SDK is waiting for a free network connection to continue the download.

  - `downloadRunning`: The download is actively in progress.

  - `updateWaiting`: An update operation is underway.

The `contentParameters` field provides information such as:

- For `roadMap` type:

    - `Copyright` : value of type `String` containing the copyright information for the road map

    - `MapData provider` : value of type `String` containing the name of the map data provider

    - `Release date` : value of type `String` containing the release date for the road map in `DD.MM.YYYY` format

    - `Default name` : value of type `String` containing the name of the item

- For `viewStyleHighRes` type:

    - `Background-Color` : value of type `String` containing the background color in decimal format (ex: `4294957738` which when converted to hex corresponds with `#FFFFDAAA`). Can be used to check if a style is a dark-mode or light-mode recommended style by checking the brightness of this value

- For `humanVoice` type:

    - `language` : value of type `String` containing the BCP 47 language code (e.g., `eng_IRL`)

    - `gender` : value of type `String` indicating the speaker's gender (e.g., `Female`)

    - `type` : value of type `String` specifying the type of voice used (e.g., `human`)

    - `native_language` : value of type `String` containing the name of the language in its native form (e.g., `English`)

The image can be got via the `imgPreview` getter:
```dart
final bool isImagePreviewAvailable = contentStoreItem.isImagePreviewAvailable;
if (isImagePreviewAvailable) {
    final Img previewImage = contentStoreItem.imgPreview;
    final Uint8List? bytes = previewImage.getRenderableImageBytes(
    size: Size(80, 80)
);
// Do something with the preview image.
}
```

Content store items of type `roadMap` do not have an image preview. The `MapDetails.getCountryFlagImg` method can be used to get the flag image associated with a country code instead.

Use the `countryCodes` getter to obtain the country codes associated with a content store item of type `roadMap`.

#### Fields Containing Download and Update Information 

| Field Name        | Description                                                       |
|-------------------|-------------------------------------------------------------------|
| `clientVersion`   | The client version of the content.                                |
| `updateVersion`   | The update version if an update is available. Returns a dummy object with fields set to 0 if no new version is available |
| `downloadProgress`| The current download progress as a percentage.                    |
| `updateItem`      | The corresponding update item if an update is in progress.        |
| `isCompleted`     | Checks if the item has been completely downloaded.                |
| `isUpdatable`     | Checks if the item has a newer version available.                 |
| `canDeleteContent`| Checks if the content can be deleted.                             |

While the download is in progress, you can retrieve details about the downloaded content:

- **`isCompleted`**: Returns `true` if the download is completed; otherwise, returns `false` (indicating that the download has not started or has started but not yet completed).

- **`downloadProgress`**: Returns the download progress as an integer between 0 and 100. It returns `0` if the download has not started.

- **`status`**: Returns the current status of the content store item.

#### Fields Relevant to `ContentType.roadMap` Type Items 

| Field Name         | Description                                                               |
|--------------------|---------------------------------------------------------------------------|
| `chapterName`      | Some large countries are divided into multiple content store items (e.g., the USA is split into states). All items within the same country share the same chapter. This chapter is empty if the item is not of the `RoadMap` type or if the country is not divided into multiple items.|
| `countryCodes`     | A list of country codes (ISO 3166-1 alpha-3) associated with the product. |
| `language`         | The full language details for the product.                                   |

#### Fields Relevant to `ContentType.humanVoice` Type Items

| Field Name         | Description                                                               |
|--------------------|---------------------------------------------------------------------------|
| `countryCodes`     | A list of country codes (ISO 3166-1 alpha-3) associated with the product. |
| `language`         | The full language details for the product.                                   |

Use the `ISOCodeConversions` class for conversions between the different types of codes. See the [internationalization documentation](../2_get-started/05_internationalization.mdx#iso-code-conversions) for more details.

## Download Content Store Item

For downloading a content store item, the `asyncDownload` method can be used:
```dart
contentStoreItem.asyncDownload(
    (error) {
        showSnackbar("Download completed with code $error");
    },
    onProgress: (progress){
        print("Download progress: ${progress.toString()} / 100");
    },
);
```

The `onComplete` is invoked at the end of the operation, returning the result of the download. If the item is successfully downloaded, err is set to `GemError.success`. In case of an error, other values may be returned (e.g., `GemError.io` if the item is already downloaded).

Additionally, users can provide an optional `onProgress` to receive real-time progress updates. This callback is triggered with the current download progress, represented as an integer between 0 and 100.

### Download on Extra Charged Networks

The SDK includes functionality to restrict downloads on networks with additional charges, which may cause downloads to not work as expected.

To enable downloads on such networks, use the `setAllowOffboardServiceOnExtraChargedNetwork` method from the `SdkSettings` class.
```dart
SdkSettings.setAllowOffboardServiceOnExtraChargedNetwork(ServiceGroupType.contentService, true);
```

Alternatively, the `asyncDownload` method can be called with `allowChargedNetworks` set to `true`, bypassing the value set via `setAllowOffboardServiceOnExtraChargedNetwork`.

If a download operation is requested while the user is on an extra-charged network, and `setAllowOffboardServiceOnExtraChargedNetwork` is set to false for the content service without passing the true value for the `allowChargedNetworks` optional parameter, the download will be automatically scheduled and will proceed once the user switches to a non-extra-charged network.
In this case, the `status` field of the corresponding `ContentStoreItem` object will be set to `ContentStoreItemStatus.downloadQueued`.

### Pause download

The download can be paused using the `pauseDownload` method. This method takes an optional callback which gets called when the pause operation is completed. In order to resume the download call `asyncDownload` as shown above.

No further operations should be performed on the `ContentStoreItem` object until the pause operation has completed and the corresponding callback has been invoked.

### Delete Downloaded Content

Downloaded content can be removed from local storage by calling the `deleteContent` method on the corresponding `ContentStoreItem` object, after checking if the item can be removed:
```dart
if (contentStoreItem.canDeleteContent){
    final error = contentStoreItem.deleteContent();
    showSnackbar("Item ${contentStoreItem.name} deletion resulted with code $error");
} else {
    showSnackbar("Item cannot be deleted");
}
```

Do not confuse the functionality provided by the `ContentStore` / `ContentStoreItem` classes with that of the `MapDownloaderService` class.

- The `ContentStore` API is designed for **downloading full offline content**, including data required for features such as free-text search, routing, and turn-by-turn navigation.

- In contrast, the `MapDownloaderService` is intended for caching map tiles mainly for visual display purposes. Tiles downloaded via `MapDownloaderService` **do not support** most search operations, routing or navigation while offline.

See the [download individual map tiles documentation](../maps/adjust-map#download-individual-map-tiles) for more details about the `MapDownloaderService`.

## Downloading overlays

Overlays can be downloaded for specific regions to enable offline functionality. To do this, you must first download a map region, after which the overlays become available for download within those offline areas. Downloading overlays for offline use is done through the `grabOverlayOfflineData` method of `OverlayService`.
```dart
final overlayUid = CommonOverlayId.safety.id; // Example overlay UID (e.g., speed cameras)
if (!OverlayService.isOverlayOfflineDataGrabberSupported(overlayUid)) {
  print('Overlay offline data grabber not supported for this overlay');
  return;
}

OverlayService.enableOverlayOfflineDataGrabber(overlayUid);

final taskHandler = OverlayService.grabOverlayOfflineData(
    uid: overlayUid,
    onComplete: (error) {
    // Handle the completion of the offline data grabber (check GemError)
  }
);
// Optionally, you can cancel the task if needed 
// OverlayService.cancelGrabOverlayOfflineData(overlayUid);

// Disable the grabber when it's no longer needed
OverlayService.disableOverlayOfflineDataGrabber(overlayUid);
```

- Not all overlays support offline data grabbing. Use the `isOverlayOfflineDataGrabberSupported` method to check if a specific overlay supports this feature.

- After completing the download, disable the offline data grabber using `disableOverlayOfflineDataGrabber` to prevent unnecessary resource usage.

After downloading, the overlay items will be available in offline mode within the downloaded map regions. You can verify if the overlay data has been successfully downloaded if overlay items are visible inside the downloaded map region in offline mode.

Ensure to enable the offline data grabber using `enableOverlayOfflineDataGrabber` before initiating the download process, otherwise, the the `onComplete` callback will return `GemError.activation` and method will return null instead of a `TaskHandler` object.

Call enableOverlayOfflineDataGrabber only with an overlay UID that supports offline grabbing; if the UID is unsupported, enabling will not work and will not return `GemError.success`.

Not all overlay types support offline functionality (eg. Alerts or Public Transit Stops). For instance, public transport stops will still require an internet connection to display the relevant data, thus will be rendered as landmarks instead of overlay items in offline mode.

You can check if the overlay data grabber has been enabled for a specific overlay using the `isOverlayOfflineDataGrabberEnabled` method.
