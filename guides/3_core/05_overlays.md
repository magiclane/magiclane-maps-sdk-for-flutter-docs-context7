---
description: Documentation for Overlays
title: Overlays
---

# Overlays

An `Overlay` is an additional map layer, either default or user-defined, with data stored on Magic Lane servers, accessible in **online** and **offline** modes, with few exceptions.

In order to define overlay data you can use the [Magic Lane Map Studio](https://developer.magiclane.com/documentation/OnlineStudio/guide_creating_a_style.html). It allows uploading data regarding the POIs and their corresponding categories and binary information (via `GeoJSON` format). An `Overlay` can have multiple associated categories and subcategories. A single item from an overlay is called an overlay item.

If the corresponding map region has been downloaded locally, overlays will not be available in offline mode, except if they are downloaded too. See the [Downloading overlays](/guides/offline/manage-content#downloading-overlays) guide for more details.
Most overlay features can be used only when a ``GemMap`` widget is created and a style containing the overlay is applied.

## OverlayInfo

### OverlayInfo structure

The class that contains information about an overlay is `OverlayInfo`. It has the following structure:

<table>
  <thead>
    <tr>
      <th>Method</th>
      <th>Description</th>
      <th>Type</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>uid</td>
      <td>Gets the unique ID of the overlay.</td>
      <td>`int`</td>
    </tr>
    <tr>
      <td>categories</td>
      <td>Gets the categories of the overlay.</td>
      <td>`List<OverlayCategory>`</td>
    </tr>
    <tr>
      <td>getCategory</td>
      <td>Gets a category by its ID.</td>
      <td>`OverlayCategory?`</td>
    </tr>
    <tr>
      <td>img</td>
      <td>Gets the image of the overlay.</td>
      <td>`Img`</td>
    </tr>
    <tr>
      <td>name</td>
      <td>Gets the name of the overlay.</td>
      <td>`String`</td>
    </tr>
    <tr>
      <td>hasCategories</td>
      <td>Checks if the category has subcategories.</td>
      <td>`bool`</td>
    </tr>
  </tbody>
</table>

### Retrieving overlays

To get all overlays which are available for the current map style, the ``OverlayService.getAvailableOverlays()`` method is used in the following way:
```dart
final Completer<GemError> completer = Completer<GemError>();
final (OverlayCollection, bool) availableOverlays = 
  OverlayService.getAvailableOverlays(onCompleteDownload: (error) {
    completer.complete(error);
});

await completer.future;
OverlayCollection collection = availableOverlays.$1;
```

The `OverlayCollection` class contains methods and getters such as:

- **size**: returns the size of the collection.

- **getOverlayAt**: returns the `OverlayInfo` at a specified index and null if it doesn't exist.

- **getOverlayById**: returns an `OverlayInfo` by a given id.

The returned `OverlayCollection` contains all the overlays available for the current map style: alerts (`CommonOverlayId.safety`), speed cameras (`CommonOverlayId.socialReports`), public transit stops (`CommonOverlayId.publicTransport`).

It is possible that information for certain overlays is unavailable and needs to be downloaded when a network connection is accessible. As a result, the method returns a record of type `(OverlayCollection, bool)`, where the boolean value in the second element is false, indicating that the information is currently unavailable. In this case, the `onCompleteDownload` callback needs to be awaited.

### Usage

The primary function of the `OverlayInfo` class is to provide the categories within an overlay. It also provides the `uid` of the overlay which can be used to filter the results of search and also can be used to toggle the visibility of the overlay on the map. Other fields can be displayed on the UI.

## OverlayCategory

The ``OverlayCategory`` class represents hierarchical data related to overlay categories.

### OverlayCategory structure

The `OverlayCategory` has the following structure:

<table>
  <thead>
    <tr>
      <th>Property / Method</th>
      <th>Description</th>
      <th>Type</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>uid</td>
      <td>The category ID.</td>
      <td>`int`</td>
    </tr>
    <tr>
      <td>overlayuid</td>
      <td>The parent overlay ID. Refers to the id of the `OverlayInfo` object</td>
      <td>`int`</td>
    </tr>
    <tr>
      <td>img</td>
      <td>The category icon.</td>
      <td>`Img`</td>
    </tr>
    <tr>
      <td>name</td>
      <td>The category name.</td>
      <td>`String`</td>
    </tr>
    <tr>
      <td>subcategories</td>
      <td>The subcategories of the category.</td>
      <td>`List<OverlayCategory>`</td>
    </tr>
    <tr>
      <td>hasSubcategories</td>
      <td>Checks if the category has subcategories.</td>
      <td>`bool`</td>
    </tr>
  </tbody>
</table>

### Usage

The `OverlayCategory` class provides the category `uid`, which can be utilized in various search functionalities to filter results. Additionally, the `uid` can be used to filter and manage overlay items that trigger alerts within the `AlarmService`.

## OverlayItem 

The `OverlayItem` represents a single element from the overlay.

### OverlayItem structure

The `OverlayItem` has the following structure:

<table>
  <thead>
    <tr>
      <th>Property / Method</th>
      <th>Description</th>
      <th>Type</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>categoryId</td>
      <td>Gets the OverlayItem's category ID. May be 0 if the item does not belong to a specfic category. Gives the id of the root category which may not be the direct parent category.</td>
      <td>int</td>
    </tr>
    <tr>
      <td>coordinates</td>
      <td>Gets the coordinates of the OverlayItem.</td>
      <td>Coordinates</td>
    </tr>
    <tr>
      <td>hasPreviewExtendedData</td>
      <td>Checks if the OverlayItem has preview extended data (dynamic data). Available only for overlays predefined.</td>
      <td>bool</td>
    </tr>
    <tr>
      <td>img</td>
      <td>Gets the image of the OverlayItem.</td>
      <td>Img</td>
    </tr>
    <tr>
      <td>name</td>
      <td>Gets the name of the OverlayItem.</td>
      <td>String</td>
    </tr>
    <tr>
      <td>uid</td>
      <td>Gets the unique ID of the OverlayItem within the overlay.</td>
      <td>int</td>
    </tr>
    <tr>
      <td>overlayInfo</td>
      <td>Gets the parent OverlayInfo.</td>
      <td>OverlayInfo</td>
    </tr>
    <tr>
      <td>previewData</td>
      <td>Gets the OverlayItem preview data as a parameters list.</td>
      <td>SearchableParameterList</td>
    </tr>
    <tr>
      <td>previewUrl</td>
      <td>Gets the preview URL for the item (if any).</td>
      <td>String</td>
    </tr>
    <tr>
      <td>overlayUid</td>
      <td>Gets the parent overlay UID.</td>
      <td>int</td>
    </tr>
    <tr>
      <td>getPreviewExtendedData</td>
      <td>Asynchronously gets the OverlayItem preview extended data.</td>
      <td>ProgressListener</td>
    </tr>
    <tr>
      <td>cancelGetPreviewExtendedData</td>
      <td>Cancels the asynchronous `getPreviewExtendedData` operation.</td>
      <td>void</td>
    </tr>
  </tbody>
</table>

The `previewData` returned is not available if the parent map tile is disposed.
Please get the preview data before further interactions with the map.

Avoid confusing the uid of `OverlayInfo`, `OverlayCategory`, and `OverlayItem`, as they each serve distinct purposes.

To check if a `OverlayItem` belongs to a `OverlayInfo`, use the `overlayUid` property, or retrieve the whole `OverlayInfo` object via the `overlayInfo` property.

The ``previewData`` getter which provides more information structured inside a ``SearchableParametersList``, information which depend on the overlay type.
We can iterate through all the parameters (which are of type ``GemParameter``) within a ``SearchableParameterList`` in the following way:
```dart
SearchableParameterList parameters = overlayItem.previewDataParameterList;
for (GemParameter param in parameters){
  // Unique for every parameter
  String? key = param.key;

  // Used for display on UI - might change depending on language
  String? name = param.name;

  // The type of param.value
  ValueType valueType = param.type;

  // The parameter value
  dynamic value = param.value;
}
```

In order to obtain preview data in a more structured way, you can use the `getPreviewParametersAs` getter, which allows you to retrieve the data as a specific class based on the overlay type. Below is an example of how to use it for different overlay types:
```dart
if (overlayItem.overlayUid == CommonOverlayId.publicTransport.id) {
  PublicTransportParameters? parameters =
      overlays.first.getPreviewParametersAs<PublicTransportParameters>();
  if (parameters == null) {
    print("Parameters are null");
    return;
  }
  String? name = parameters.name;
  DateTime? createStamp = parameters.createStampUtc;
  int? iconId = parameters.iconId;
  bool? streetDirectionFlag = parameters.strDrivingDirectionFlag;
}
if (overlayItem.overlayUid == CommonOverlayId.safety.id) {
  SafetyParameters? parameters =
      overlays.first.getPreviewParametersAs<SafetyParameters>();
  if (parameters == null) {
    print("Parameters are null");
    return;
  }
  String? countryName = parameters.country;
  int? speedValue = parameters.speedValue;
  // other fields...
}
if (overlayItem.overlayUid == CommonOverlayId.socialReports.id) {
  SocialReportParameters? parameters =
      overlays.first.getPreviewParametersAs<SocialReportParameters>();
  if (parameters == null) {
    print("Parameters are null");
    return;
  }
  int? title = parameters.score;
  String? description = parameters.description;
  DateTime? createStamp = parameters.createStampUtc;
  // other fields...
}
```

The `categoryId` getter returns the ID of the root category, which may not necessarily be the direct parent category of the `OverlayItem`.

To obtain the direct parent category, you can use the following snippet:
```dart
final int parentCategoryId = overlayItem.previewDataParameterList.findParameter('icon').value as int;
```

Use the `getCategory` method from the parent `OverlayInfo` class to retrieve the `OverlayCategory` object corresponding to this ID.

### Usage

``OverlayItems`` can be selected from the map or be provided by the ``AlarmService`` on approach.  Other fields and information can be displayed on the UI.

## Classification

There are a few types of predefined overlays:

  - Safety overlay

  - Public transport overlay

  - Social reports overlay

The `CommonOverlayId` enum contains information about the ids of the predefined overlay categories. 

### Safety Overlay

These overlays represent speed limit cameras, red light controls and so on.

For a speed limit, ``previewData`` might include information searchable by keys like:

- ``eStrDrivingDirection``: the driving direction on which the overlay item applies as string, eg. "Both Ways"

- ``speedValue``: value of maximum allowed speed as integer, eg. 50

- ``speedUnit``: measure unit of speed as string, eg "km/h"

- ``Country``: country name where the overlay is reported as string, eg "FRA"

### Public Transport Overlay

This overlay is responsible for displaying public transport stations.

For a bus station, ``previewData`` might include information searchable by keys like:

- ``id``: unique overlay id as integer

- ``create_stamp_utc``: unix epoch time when overlay has been created as integer

- ``icon``: an array of 8-bit integeres representing icon data as Uint8List

- ``name``: name of bus station as a string

There are two types of public transport stops available on the map:

- Bus station with schedule information, available on the map as overlay items.

- Bus station without schedule information, available on the map as landmarks.

### Social Reports Overlay

This overlay is responsible for displaying fixed cameras, construction sites and more.

For a construction report, ``previewData`` includes information searchable by keys like:

- ``longitude``: value for longitude coordinate as double

- ``latitude``: value for latitude coordinates as double

- ``owner_name``: user's name that reported the event as string

- ``score``: likes (confirmations) offered by other users

- ``location_address``: address of reported location as string, it contains street, city and country

The ``SocialOverlay`` static class is responsible for generating, updating, and deleting social reports. It includes several static methods designed to perform these operations efficiently.

## Interaction with Overlays

### Selecting overlay items

Overlay items are selectable. When user taps or clicks, you can identify specific overlay items programmatically (e.g., through the function `cursorSelectionOverlayItems()`). Please refer to the [Map Selection Functionality](/guides/maps/interact-with-map#map-selection-functionality) guide for more details.

### Searching overlay items

Overlays are searchable. This can be done in multiple ways, the most common being to set the right properties in the search preferences when performing a regular search. More details can be found within the [Get started with Search](/guides/search/get-started-search) guide.

### Calculating route with overlay items

Overlay items are **not** designed for route calculation and navigation.

Create a new landmark using the overlay item's relevant coordinates and a representative name, then utilize this object for routing purposes.

### Get notifications when approaching overlay items

Alarms can be configured to notify users when they approach specific overlay items from selected overlays.
See the [Landmarks and overlay alarms](/guides/alarms/landmark-and-overlay-alarms) guide for more details about implementing this feature.

### Activate highlights

Overlay Items can be highlighted using the `activateHighlightOverlayItems` method provided by the `GemMapController` class.
