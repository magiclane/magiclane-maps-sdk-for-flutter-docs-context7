---
description: Documentation for Overlays
title: Overlays
---

# Overlays

An `Overlay` is an additional map layer with data stored on Magic Lane servers, accessible in online and offline modes. Overlays can be default or user-defined.

Define overlay data using [Magic Lane Map Studio](https://developer.magiclane.com/documentation/OnlineStudio/guide_creating_a_style.html). Upload POI data, categories, and binary information via `GeoJSON` format. Overlays can have multiple categories and subcategories. A single item from an overlay is an overlay item.

Overlays require downloading to work offline. See [Downloading overlays](/guides/offline/manage-content#download-overlays) for details. Most overlay features require a `GemMap` widget with a style containing the overlay.

---

## OverlayInfo

The `OverlayInfo` class contains information about an overlay.

### Structure

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

### Usage

Use `OverlayInfo` to:

- Access categories within an overlay

- Get the overlay `uid` for filtering search results

- Toggle overlay visibility on the map

- Display overlay information in the UI

---

## OverlayCategory

The `OverlayCategory` class represents hierarchical data for overlay categories.

### Structure

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

Use the category `uid` to:

- Filter search results

- Filter and manage overlay items that trigger alerts in `AlarmService`

---

## OverlayItem

An `OverlayItem` represents a single item within an overlay, containing information about the item and its parent overlay.

### Structure

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
      <td>Gets the OverlayItem's category ID. May be 0 if the item does not belong to a specific category. Gives the id of the root category which may not be the direct parent category.</td>
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

Don't confuse the `uid` of `OverlayInfo`, `OverlayCategory`, and `OverlayItem`—each serves a distinct purpose.

Check if an `OverlayItem` belongs to an `OverlayInfo` using the `overlayUid` property, or retrieve the full `OverlayInfo` object via the `overlayInfo` property.

The `categoryId` getter returns the root category ID, not necessarily the direct parent category.

Get the direct parent category:
```dart
final int parentCategoryId = overlayItem.previewDataParameterList.findParameter('icon').value as int;
```

Use the `getCategory` method from the parent `OverlayInfo` class to retrieve the corresponding `OverlayCategory` object.

### Usage

Select `OverlayItems` from the map or receive them from `AlarmService` on approach. Display overlay item fields and information in the UI.

---

## Overlay Types

Predefined overlay types:

- Safety overlay

- Public transport overlay

- Social reports overlay

The `CommonOverlayId` enum contains IDs for predefined overlay categories. 

### Safety Overlay

Safety overlays represent speed limit cameras, red light controls, and similar items.

Speed limit `previewData` includes:

- `eStrDrivingDirection` - Driving direction (e.g., "Both Ways")

- `speedValue` - Maximum allowed speed (e.g., 50)

- `speedUnit` - Speed unit (e.g., "km/h")

- `Country` - Country code (e.g., "FRA")

### Public Transport Overlay

This overlay displays public transport stations.

Bus station `previewData` includes:

- `id` - Unique overlay ID

- `create_stamp_utc` - Unix epoch creation time

- `icon` - Icon data as `Uint8List`

- `name` - Bus station name

Two types of public transport stops exist:

- Bus stations with schedule information (overlay items)

- Bus stations without schedule information (landmarks)

### Social Reports Overlay

This overlay displays fixed cameras, construction sites, and user-reported events.

Construction report `previewData` includes:

- `longitude` - Longitude coordinate

- `latitude` - Latitude coordinate

- `owner_name` - User who reported the event

- `score` - Confirmations from other users

- `location_address` - Location address (street, city, country)

The `SocialOverlay` static class generates, updates, and deletes social reports through static methods.

---

## Work with Overlays

### OverlayService

The `OverlayService` manages overlays in the Maps SDK, providing methods to retrieve, enable, disable, and manage overlay data online and offline.

#### Retrieve Overlay Information

Retrieve all available overlays for the current map style using `getAvailableOverlays`.

This method returns an `(OverlayCollection, bool)` tuple:

- `OverlayCollection` - Contains available overlays

- `bool` - Indicates if some information is unavailable and will download when network is available

Receive a notification when missing information downloads via the `onCompleteDownload` callback.
```dart
final Completer<GemError> completer = Completer<GemError>();
final (OverlayCollection, bool) availableOverlays = 
  OverlayService.getAvailableOverlays(onCompleteDownload: (error) {
    completer.complete(error);
});

await completer.future;
OverlayCollection collection = availableOverlays.$1;
```

The `OverlayCollection` class provides:

- `size` - Returns collection size

- `getOverlayAt` - Returns `OverlayInfo` at specified index (null if it doesn't exist)

- `getOverlayById` - Returns `OverlayInfo` by ID

#### Enable and Disable Overlays

Enable or disable overlays using `enableOverlay` and `disableOverlay` methods. Check overlay status with `isOverlayEnabled`.
```dart
final int overlayUid = CommonOverlayId.safety.id;

// Enable overlay
final GemError errorCodeWhileEnabling = OverlayService.enableOverlay(overlayUid);

// Disable overlay
final GemError errorCodeWhileDisabling = OverlayService.disableOverlay(overlayUid);

// Check if overlay is enabled
final bool isEnabled = OverlayService.isOverlayEnabled(overlayUid);
```

The `enableOverlay`, `disableOverlay`, and `isOverlayEnabled` methods can also take an optional `categUid` parameter to enable, disable, or check the status of a specific category within an overlay.
By default, if no category ID is provided, the entire overlay is affected.

## Select overlay items

Overlay items are selectable. Identify specific items programmatically when users tap or click using `cursorSelectionOverlayItems()`. See [Map Selection Functionality](/guides/maps/interact-with-map#select-map-elements) for details.

### Search Overlay Items

Overlays are searchable. Set the right properties in search preferences when performing a search. See [Get started with Search](/guides/search/get-started-search) for details.

### Calculate Routes

Overlay items are **not designed for route calculation**.

For routing, create a landmark using the overlay item's coordinates and a representative name.

### Display Overlay Item Information

Overlay items contain additional information for display. Access this information using:

- `previewDataParameterList` getter

- `getPreviewParametersAs` method

- `previewUrl` getter (returns a URL for more details in a web browser)

The `previewData` getter provides information structured in a `SearchableParametersList`, varying by overlay type. Iterate through parameters (type `GemParameter`):
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

The `previewData` is unavailable if the parent map tile is disposed. Get preview data before further map interactions.

Obtain structured preview data using the `getPreviewParametersAs` getter, which retrieves data as a specific class based on overlay type:
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

Retrieve the image associated with an overlay item using the `img` property.

### Proximity Alarms

Configure alarms to notify users when approaching specific overlay items. See [Landmarks and overlay alarms](/guides/alarms/landmark-and-overlay-alarms) for implementation details.

### Highlight Overlay Items

Highlight overlay items using the `activateHighlightOverlayItems` method from the `GemMapController` class.

---
