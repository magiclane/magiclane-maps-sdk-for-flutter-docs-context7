---
description: Documentation for Styling
title: Styling
---

# Styling

The appearance of the map can be tailored by applying different styles. You can either download a predefined map style using the ``ContentStore`` class, which offers a variety of ready-to-use styles, or create a custom style using [Magic Lane Map Studio](https://developer.magiclane.com/documentation/OnlineStudio/guide_creating_a_style.html) which you can download and configure. In this guide, we’ll explore both methods in detail.

## Apply predefined styles

To apply a predefined map style, it must first be downloaded, as it is not loaded into memory by default. As mentioned earlier, this can be achieved using the ``ContentStore`` class. To begin, we’ll retrieve a list of all available styles for preview purposes and then proceed to download the ones we wish to use.

Here’s how you can get previews of the available map styles, represented as a ``List<ContentStoreItem>``, with the following code:
```dart
void getStyles() {
  ContentStore.asyncGetStoreContentList(ContentType.viewStyleLowRes,
      (err, items, isCached) {
    if (err == GemError.success && items != null) {
      for (final item in items) {
        _stylesList.add(item);
      }
      ScaffoldMessenger.of(context).clearSnackBars();
    }
  });
}
```

Method ``asyncGetStoreContentList`` can be used to obtain other content such as car models, road maps, tts voices and more.

There are two types of preview styles available: ``ContentType.viewStyleHighRes`` and ``ContentType.viewStyleLowRes.``

- ``ContentType.viewStyleHighRes`` is designed for obtaining styles optimized for high-resolution displays, such as those on mobile devices.

- ``ContentType.viewStyleLowRes`` is intended for styles suited to low-resolution displays, such as desktop monitors.

In the ``onComplete`` parameter of the ``asyncGetStoreContentList`` method, several values are provided:

- ``GemError`` object that indicates whether any errors occurred during the operation.

- ``List<ContentStoreItem>`` that contains the items retrieved from the content store, such as map styles in this case.

- boolean value that specifies whether the content store item (e.g., the map style) is already available in cache memory (and thus doesn't require downloading) or if it needs to be downloaded.

A ``ContentStoreItem`` has the following attributes/methods:

| Attribute/Methods                   | Explanation                                                                 |
|-------------------------------------|-----------------------------------------------------------------------------|
| name                                | Gets the name of the associated product.                                    |
| id                                  | Get the unique id of the item in the content store.                         |
| chapterName                         | Gets the product chapter name translated to interface language.             |
| countryCodes                        | Gets the country code (ISO 3166-1 alpha-3) list of the product as text.     |
| language                            | Gets the full language code for the product.                                |
| type                                | Gets the type of the product as a [ContentType] value.                      |
| fileName                            | Gets the full path to the content data file when available.                 |
| clientVersion                       | Gets the client version of the content.                                     |
| totalSize                           | Get the size of the content in bytes.                                       |
| availableSize                       | Gets the available size of the content in bytes.                            |
| isCompleted                         | Checks if the item is completed downloaded.                                 |
| status                              | Gets current item status.                                                   |
| pauseDownload                       | Pause a previous download operation.                                        |
| cancelDownload                      | Cancel a previous download operation.                                       |
| downloadProgress                    | Get current download progress.                                              |
| canDeleteContent                    | Check if associated content can be deleted.                                 |
| deleteContent                       | Delete the associated content                                               |
| isImagePreviewAvailable             | Check if there is an image preview available on the client.                 |
| imgPreview                          | Get the preview. The user is responsible to check if the image is valid.    |
| contentParameters                   | Get additional parameters for the content.                                  |
| updateItem                          | Get corresponding update item.                                              |
| isUpdatable                         | Check if item is updatable, i.e. it has a newer version available.          |
| updateSize                          | Get update size (if an update is available for this item).                  |
| updateVersion                       | Get update version (if an update is available for this item).               |
| asyncDownload                       | Asynchronous start/resume the download of the content store product content.|

Keep in mind that certain attributes may not apply to specific types of ``ContentStoreItem``. For instance, the ``countryCodes`` attribute will not provide meaningful data for a ``ContentType.viewStyleLowRes``, as styles are not associated with any particular country.

Downloading a map style is done by calling ``ContentStoreItem.asyncDownload()`` as shown below:
```dart
Future<bool> _downloadStyle(ContentStoreItem style) async {
  setState(() {
    _isDownloadingStyle = true;
  });
  Completer<bool> completer = Completer<bool>();
  style.asyncDownload((err) {
    if (err != GemError.success) {
      // An error was encountered during download
      completer.complete(false);
      setState(() {
        _isDownloadingStyle = false;
      });
      return;
    }
    // Download was successful
    completer.complete(true);
    setState(() {
      _isDownloadingStyle = false;
    });
  }, onProgressCallback: (progress) {
    // Gets called every time download progresses with a value between [0, 100]
    print('progress: $progress');
  }, allowChargedNetworks: true);
  return await completer.future;
}
```

Now, all that is left to do is applying the downloaded style by using ``GemMapController.MapViewPreferences.setMapStyleByPath(path)`` called with the filename, which contains the path:
```dart
final String filename = currentStyle.fileName;
mapController.preferences.setMapStyleByPath(filename);
```

To wrap things up, this is the code that incorporates all steps:
```dart
if (_stylesList.isEmpty) {
    _showSnackBar(context, message: "The map styles are loading.");
    getStyles();
    return;
}

final indexOfNextStyle = (_indexOfCurrentStyle >= _stylesList.length - 1)
    ? 0
    : _indexOfCurrentStyle + 1;
ContentStoreItem currentStyle = _stylesList[indexOfNextStyle];

if (currentStyle.isCompleted == false) {
  final didDownloadSuccessfully = await _downloadStyle(currentStyle);
  if (didDownloadSuccessfully == false) return;
}

_indexOfCurrentStyle = indexOfNextStyle;

final String filename = currentStyle.fileName;
mapController.preferences.setMapStyleByPath(filename);
```

Map styles can be set by using ``MapViewPreferences.setMapStyle()`` or ``MapViewPreferences.setMapStyleById()``.

- ``MapViewPreferences.setMapStyle()`` takes as parameter the ``ContentStoreItem`` which needs to be of type ``ContentType.viewStyleHighRes`` or ``ContentType.viewStyleLowRes``

- ``MapViewPreferences.setMapStyleById()`` takes as parameter the unique id of the ``ContentStoreItem``, obtained by calling ``ContentStoreItem.id``
```dart
mapController.preferences.setMapStyle(currentStyle);
mapController.preferences.setMapStyleById(currentStyle.id);
```

## Apply custom styles

A custom map style can be created in [Magic Lane Map Studio](https://developer.magiclane.com/documentation/OnlineStudio/guide_creating_a_style.html). By following the guide you'll end up with a .style file. This file will be loaded into application and applied as a style.

We need to create an ``assets`` directory in the root of the project, where the .style file will be placed. Next, the following lines will be added to the ``pubspec.yaml`` file, under ``flutter:`` section, as shown below:
```yaml
flutter:
  uses-material-design: true

  assets:
    - assets/
```

This is necessary for the ``flutter/services.dart`` package to have access to the ``assets`` directory.

Loading the style into memory is done with the following code:
```dart
// Method to load style and return it as bytes
Future<Uint8List> _loadStyle() async {
  // Load style into memory
  final data = await rootBundle.load('assets/Basic_1_Oldtime-1_21_656.style');

  // Convert it to Uint8List
  final bytes = data.buffer.asUint8List();

  return bytes;
}
```

In order for the ``rootBundle.load()`` to work, ``'package:flutter/services.dart'`` needs to be imported.

Once the map style bytes are obtained, the style can be set by using ``MapViewPreferences.setMapStyleByBuffer(styleData)``:
```dart
final styleData = await _loadStyle();

mapController.preferences
    .setMapStyleByBuffer(styleData, smoothTransition: true);
```

A smooth transition can be enabled by passing the ``smoothTransition`` parameter of ``setMapStyleByBuffer`` as true.

In order have a map style already applied when creating a ``GemMap``, a relative path to the .style file is provided to the optional parameter ``initialMapStyleAsset``. The following code explains the process:
```dart
GemMap(
  appAuthorization: projectApiToken,
  initialMapStyleAsset: "assets/map-styles/my_map_style.style",
),
```

While using the ``initialMapStyleAsset`` parameter the path of the style file is relative to the project root and only works on Android and iOS.

## Get notified about style changes

The user can be notified when the style changes by providing a callback using the `registerSetMapStyleCallback` method from the `GemMapController`:
```dart
controller.registerSetMapStyleCallback((id, stylepath, viaApi){
  print('The style with id $id and path $stylepath has been set. viaApi: $viaApi');
});
```

The callback provides the following parameters:

- `id`: The id of the style

- `stylepath`: The path to the `.style` file

- `viaApi`: A boolean indicating if the style was set via API or not

## Relevant examples demonstrating map styling related features

- [Map Styles](/examples/maps-3dscene-examples/map-styles)

- [Assets Map Style](/examples/maps-3dscene-examples/assets-map-styles)
