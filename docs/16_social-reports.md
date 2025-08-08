---
description: Documentation for Social Reports
title: Social Reports
---

# Social reports

Social reports are user-generated alerts about real-time driving conditions or incidents on the road. These reports can include various types of information, such as accidents, police presence, road construction and more.

Users can create new reports, with the possibility to provide information such as category, name, image and other parameters. They can provide feedback, enabling voting on the accuracy of incidents and commenting on reported events. They can confirm or deny the validity of reports, delete their own reports, and contribute additional comments for further context. These interactions help enhance the accuracy and reliability of the information shared within the community.

Social reports are visible by all users with the social overlay enabled, given a compatible map style.

## Report categories

The following categories and subcategories are provided in the form of a hierarchical structure, based on the `OverlayCategory` class:
```
┌ Police Car (id 256)
│   - My Side (id 264)
│   - Opposite Side (id 272)
│   - Both Sides (280)
└■
┌ Fixed Camera (id 512)
│   - My Side (id 520)
│   - Opposite Side (id 528)
│   - Both Sides (536)
└■
┌ Traffic (id 768)
│   - Moderate (id 776)
│   - Heavy (id 784)
│   - Standstill (792)
└■
┌ Crash (id 1024)
│   - My Side (id 1032)
│   - Opposite Side (id 1040)
└■
┌ Crash (id 1024)
│   - My Side (id 1032)
│   - Opposite Side (id 1040)
└■
┌ Road Hazard (id 1280)
│   - Pothole (id 1288)
│   - Constructions (id 1296)
│   - Animals (id 1312)
│   - Object on Road (id 1328)
│   - Vehicle Stopped on Road (id 1344)
└■
┌ Weather Hazard (id 1536)
│   - Fog (id 1544)
│   - Ice on Road (id 1552)
│   - Flood (id 1560)
│   - Hail (id 1568)
└■
┌ Road Closure (id 3072)
│   - My Side (id 3080)
│   - Opposite Side (id 3088)
│   - Both Sides (id 3096)
└■
```

The main categories and subcategories can be retrieved via the following snippet:
```dart
final List<OverlayCategory> categories = SocialOverlay.reportsOverlayInfo.categories;

for (final OverlayCategory category in categories){
    print("Category name: ${category.name}");
    print("Category id: ${category.uid}");
    
    for (final OverlayCategory subCategory in category.subcategories){
        print("Subcategory name: ${subCategory.name}");
        print("Subcategory id: ${subCategory.uid}");
    }
}
```

More details about the `OverlayCategory` class structure can be found in the [Overlay documentation](/guides/core/overlays).

## Uploading a Social Report

Before uploading a social report, it must first be prepared. The ``SocialOverlay`` class provides methods like ``prepareReporting`` and ``prepareReportingCoords`` to handle the report preparation phase. 

The ``prepareReporting`` method takes a category ID and uses the current user's location, while ``prepareCoordinates`` accepts both a category ID and a ``Coordinates`` entity, enabling reporting from a different location. Those methods return an integer, called ``prepareId`` which is later passed to ``report`` method, in order to upload a social overlay item.

The following code snippet performs prepare and report of a social overlay item:
```dart
// Get the reporting id (uses current position)
int idReport = SocialOverlay.prepareReporting(categId: 0);

// Get the subcategory id
SocialReportsOverlayInfo info = SocialOverlay.reportsOverlayInfo;
List<SocialReportsOverlayCategory> categs = info.getSocialReportsCategories();
SocialReportsOverlayCategory cat = categs.first;
List<SocialReportsOverlayCategory> subcats = cat.overlaySubcategories;
SocialReportsOverlayCategory subCategory = subcats.first;

// Report
EventHandler? handler = SocialOverlay.report(
    prepareId: idReport,
    categId: subCategory.uid,
    onComplete: (error) {
        print("Report result error: $error");
    },
);
```

The report is displayed for a limited duration before being automatically removed.

The report result will be provided via the `onComplete` callback, with the following `GemError` values:

- `invalidInput` if the category id is invalid/ the parameters are ill formatted or if the snapshot is an invalid image.

- `suspended` if the rate limit for the user is exceeded.

- `expired` if the prepared report is too old.

- `notFound` if no accurate data source is detected.

- `success` if the operation has succeeded

The method returns an `EventHandler` instance, which can be used to cancel the operation by calling the static `cancel` method from the `SocialOverlay` class (operation applicable also for other operations such as upvote, downvote, update, etc.).  
If the operation could not be started, the method returns `null`.

Most report categories require the use of the `prepareReporting` method, ensuring higher report accuracy by confirming the user’s proximity to the reported location. See the [Get started with Positioning](/guides/positioning/get-started-positioning) guide for more information about configuring the data source.

The `prepareReportingCoords` method works only for `Weather Hazard` categories and subcategories contained within.

While reporting events, the `prepareReporting` method needs to be in preparing mode (categId=0) rather than dry run mode (categId !=0).

The `report` function accepts the following optional parameters:

- `snapshot` and `format`:  Used to provide an image for the report. For example, `snapshot` may refer to a file path or image data, and `format` could specify the image type (e.g., `"png"` or `"jpeg"`).

- `params`:  A `ParameterList` configuration object that allows further customization of the report details.

These parameters are optional and can be omitted if not needed.

## Updating a Social Report

In order to update an existing report's parameters the ``SocialOverlay.updateReport(item, params)`` method can be used as shown below:
```dart
List<OverlayItem> overlays = mapController.cursorSelectionOverlayItems();

SearchableParameterList params = overlays.first.previewData;
GemParameter param = params.findParameter("location_address");
param.value = "New address";

final handler = SocialOverlay.updateReport(
    item: overlays.first,
    params: params,
    onComplete: (GemError error) {
        print("Update result error: $error");
    },
);
```

The structure of the `SearchableParameterList` object passed to the `update` method should follow the structure returned by the `OverlayItem`'s `previewData`. The keys of the fields accepted can be found inside `PredefinedOverlayGenericParametersIds` and `PredefinedReportParameterKeys`. 

The `report` method might provide the following `GemError` values via the `onComplete` callback:

- `invalidInput` if the `SearchableParameterList`'s structure is incorrect.

- `success` if the operation has been successfully completed.

A user can obtain a report `OverlayItem` through the following methods:

- **Map Selection**: By selecting an item directly from the map using the `cursorSelectionOverlayItems` method provided by the `GemMapController`.

- **Search**: By performing a search that includes preferences configured to return overlay items.

- **Proximity Alerts**: Via the `AlarmListener`, when approaching a report that triggers an alert.

## Deleting a Social Report

This is accomplished through ``SocialOverlay.deleteReport(overlayItem)``, with the restriction that only the original creator of the report has the authority to delete it.

The `delete` method might provide the following `GemError` values on the `onComplete` method:

- `invalidInput` if the item is not a social report overlay item or not the result of an alarm notification.

- `accessDenied` if the user does not have the required rights.

- `success` if the operation was successfully completed.

## Interact with a Social Report

### Provide positive feedback

Users can provide positive feedback for a reported event using ``SocialOverlay.confirmReport()``, which increases the value of the score key within ``OverlayItem.previewData``.

### Provide negative feedback

If a report is found to be inaccurate, it can be denied by other users using ``SocialOverlay.denyReport()``, which accepts an ``OverlayItem`` object as its parameter.

If a report gets many downvotes it will be removed.

Both `confirmReport` and `denyReport` will provide the following `GemError` values using the `onComplete` callback:

- `invalidInput` if the item is not a social report overlay item or not the result of an alarm notification.

- `accessDenied` if the user already voted.

- `success` if the operation was successfully completed.

### Add comment

Additionally, users can contribute comments to a reported event by calling ``SocialOverlay.addComment``, as shown below:
```dart
final handler = SocialOverlay.addComment(
    item: overlay,
    comment: "This is a comment",
    onComplete: (GemError error) {
        print("Add comment result error: $error");
    },
);
```

Added comments can be viewed within the `OverlayItem`'s `previewData`

The `addComment` method will provide the following `GemError` values on the `onComplete` callback:

- `invalidInput` if the item is not a social report overlay item or not the result of an alarm notification.

- `connectionRequired` if no internet connection is available.

- `busy` if another comment operation is in progress.

- `success` if the comment was added.

## Get updates about a report

A user may be interested in tracking changes to a report—whether they made changes or not. For this purpose, the `SocialReportListener` class can be used.

To create and register a listener for a specific `OverlayItem` report, refer to the following snippet:
```dart
SocialReportListener listener = SocialReportListener(
    onReportUpdated: (OverlayItem report) {
    print('The report has been updated');
    },
);

GemError error = SocialOverlay.registerReportListener(overlay, listener);
if (error != GemError.success) {
    print('The register failed');
}
```

The `registerReportListener` method returns the following possible values:

- `GemError.success` : Listener successfully registered.

- `GemError.invalidInput` : Provided `OverlayItem` is not a social overlay item.

- `GemError.exist` : Listener already registered for the report.

In order to unregister the listener:
```dart
GemError error = SocialOverlay.unregisterReportListener(overlay, listener);
if (error != GemError.success) {
    print('The unregister failed');
}
```

The `unregisterReportListener` method returns the following possible values:

- `GemError.success` : Listener successfully removed.

- `GemError.invalidInput` : Provided `OverlayItem` is not a social overlay item.

- `GemError.notFound` : Listener was not registered for the report.
