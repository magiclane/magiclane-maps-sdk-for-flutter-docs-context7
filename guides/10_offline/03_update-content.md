---
description: Documentation for Update Content
title: Update Content
---

# Update Content

The Magic Lane SDK allows updating downloaded content to stay synchronized with the latest map data.

New map versions are released every few weeks. The update operation supports the `roadMap`, `viewStyleLowRes`, and `viewStyleHighRes` content types. This guide focuses on the `roadMap` type, as it is the most common use case.

The SDK requires all road map content store items to have the same version. Partial updates of individual items are not supported.

## Content store status

Based on the client's content version relative to the newest available release, it can be in one of three states, as defined by the `ContentStoreStatus` enum:

| Status        | Description  |
|---------------|--------------|
| `expiredData` | The client version is significantly outdated and no longer supports online operations.  All features—such as search, route calculation, and navigation—will function exclusively on the device using downloaded regions, even if an internet connection is available. If an operation like route calculation is attempted in non-downloaded regions, it will fail with a `GemError.expired` error code. An update is **mandatory** to restore online functionality. Only relevant for `ContentType.roadMap` elements. |
| `oldData`     | The client version is outdated but still supports online operations.  Features will work online when a connection is available and offline when not. While offline, only downloaded regions are accessible, but online access allows operations across the entire map.  An update is **recommended** as soon as possible. Relevant for all types of content store elements.|
| `upToDate`    | The client version has the latest map data. Features function online when connected and offline using downloaded regions.  No updates are available. Relevant for all types of content store elements. |

The Magic Lane servers support online operations for the most up-to-date version (`ContentStoreStatus.upToDate`) and the previous version (`ContentStoreStatus.oldData`) of road map data.

The Magic Lane Flutter SDK is designed for seamless automatic updates by default, ensuring compatibility with the latest map data with minimal effort from the API user. If manual map update management is not required for your use case, no additional configuration is needed.

After installation, the app includes a default map version of type `expiredData`, which contains no available content. An update—either manually triggered or performed automatically—is required before the app can be used. Internet access is required for the initial use of the app.

---

## Update process overview

The update process follows these steps:

1. The map update process is initiated by the API user or the SDK automatically starts the download process
2. The process downloads the newer data in background ensuring the full usability of the current (old) map dataset for browsing, search and navigation. The content is downloaded in a close-to-current user position order—nearby maps are downloaded first
3. Once all new version data is downloaded:

    - If the auto-update feature is enabled, the update is automatically applied

    - If the user initiated the update manually, the API user is notified and the update is applied by replacing the files in an atomic operation

If the storage size does not allow the existence of both the old and new dataset at the same time, an additional step is required:

4. The remaining offline maps that did not download because of the out-of-space exception should be downloaded by calling `ContentStoreItem.asyncDownload`

The auto-update behavior is different between the Magic Lane SDKs:

- The C++ SDK does not provide an auto-update mechanism

- The Kotlin SDK provides an auto-update mechanism

- The iOS SDK does not provide an auto-update mechanism

---

## Listen for auto-update completion

Use the `registerOnAutoUpdateComplete` method from the `OffBoardListener` class to listen for auto-update completion events.
```dart
SdkSettings.offBoardListener.registerOnAutoUpdateComplete((ContentType type, GemError error) {
    if (error == GemError.success) {
        print("The update process finished successfully for $type");
    } else {
        print("The update process failed for $type! The error code is $error");
    }
});
```

The callback is triggered for each content type when the auto-update process finishes (only for types configured to auto-update).

If the auto-update fails, you are responsible for handling this case and triggering an update manually if needed.

---

## Configure automatic updates

By default, automatic updates are enabled for road maps and styles. Configure this behavior using the `AutoUpdateSettings` class, which can be passed to the `GemKit.initialize` method at the start of the application.

Automatic updates can be customized individually for each content type:
```dart
AutoUpdateSettings settings = AutoUpdateSettings(
    isAutoUpdateForRoadMapEnabled: false,
    isAutoUpdateForViewStyleHighResEnabled: true,
    isAutoUpdateForViewStyleLowResEnabled: false,
);

await GemKit.initialize(
    appAuthorization: "YOUR_API_TOKEN",
    autoUpdateSettings: settings);
```

You can also configure auto-update settings later in the application using the `OffBoardListener` class:
```dart
// Via the AutoUpdateSettings object
SdkSettings.offBoardListener.autoUpdateSettings = AutoUpdateSettings(
    isAutoUpdateForRoadMapEnabled: true,
    isAutoUpdateForViewStyleHighResEnabled: false,
    isAutoUpdateForViewStyleLowResEnabled: false,
);

// Via the setters
SdkSettings.offBoardListener.isAutoUpdateForRoadMapEnabled = true;
```

If you change auto-update settings via the `SdkSettings.offBoardListener` object, call `SdkSettings.autoUpdate()` to check for new updates and automatically download and apply them.

If the update has already been completed for a specific type, the auto-update configuration for that type is no longer taken into account. You cannot return to an older version once an update has been applied.

The `AutoUpdateSettings` class also includes the `AutoUpdateSettings.allDisabled()` and `AutoUpdateSettings.allEnabled()` constructors for quickly disabling or enabling all updates.

---

## Listen for map updates

Listen for map updates by calling the `registerOnWorldwideRoadMapSupportStatus` method and providing a callback.
```dart
SdkSettings.offBoardListener.registerOnWorldwideRoadMapSupportStatus((status){
        switch (status) {
          case ContentStoreStatus.upToDate:
            showSnackbar("The map version is up-to-date.");
            break;
          case ContentStoreStatus.oldData:
            showSnackbar(
                "A new map version is available. Online operation on the current map version are still supported.");
            break;
          case ContentStoreStatus.expiredData:
            showSnackbar(
                "The map version has expired. All operations will be executed offline.");
            break;
        }
      },
    );
```

The SDK may automatically trigger the map version check at an appropriate moment. To manually force the check, call the `checkForUpdate` method from the `ContentStore`:
```dart
final GemError checkUpdateCode = ContentStore.checkForUpdate(ContentType.roadMap);
print('Check for update result code $checkUpdateCode');
```

The `checkForUpdate` method returns `GemError.success` if the check has been initiated and `GemError.connectionRequired` if no internet connection is available.

If the `checkForUpdate` method is provided with the `ContentType.roadMap` argument, the `onWorldwideRoadMapSupportStatusCallback` will be called. If other values are supplied (such as map styles), the response will be returned via the `onAvailableContentUpdateCallback` callback.

---

## Create a content updater

Instantiate a `ContentUpdater` object to update road maps. This object manages all operations related to the update process:
```dart
final (contentUpdater, contentUpdaterCreationError) = ContentStore.createContentUpdater(ContentType.roadMap);

if (contentUpdaterCreationError != GemError.success && contentUpdaterCreationError != GemError.exist){
    print("Error regarding the content updater creation : $contentUpdaterCreationError");
    return;
}
```

The `createContentUpdater` method returns a `ContentUpdater` instance along with an error code indicating the status of the updater creation for the specified `ContentType`:

- If the error code is `GemError.success`, the `ContentUpdater` was successfully created and is ready for use

- If the error code is `GemError.exist`, a `ContentUpdater` for the specified `ContentType` already exists, and the existing instance is returned. This instance remains valid and can be used

- If the error code corresponds to any other `GemError` value, the `ContentUpdater` instantiation has failed, and the returned object is not usable

---

## Start the update

Call the `update` method from the `ContentUpdater` object to start the update process.

The `update` method accepts the following parameters:

- A boolean indicating whether the update can proceed on networks that may incur additional charges. If false, the update will only run on free networks, such as Wi-Fi

- A callback named `onStatusUpdated` that is triggered when the update status changes, providing information through the `ContentUpdaterStatus` parameter

- A callback named `onProgressUpdated` that is triggered when the update progresses, supplying an integer value between 0 and 100 to indicate completion percentage

- A callback named `onComplete` that is triggered with the update result at the end of the update process (after calling `apply` or if the update fails earlier). The most common error codes are:

    - `GemError.success` if the update was successful

    - `GemError.inUse` if the update is already running and could not be started

    - `GemError.notSupported` if the update operation is not supported for the given content type

    - `GemError.noDiskSpace` if there is not enough space on the device for the update

    - `GemError.io` if an error regarding file operations occurred

The method returns a non-null `ProgressListener` if the update process has been successfully started or `null` if the update couldn't be started (in which case the `onComplete` will be called with the error code). 
```dart
final ProgressListener? listener = contentUpdater.update(
    true, // <-  Allow update on charge network
    onStatusUpdated: (ContentUpdaterStatus status) {
        print("OnStatusUpdated with code $status");
    },
    onProgressUpdated: (progress) {
        print("Update progress $progress/100");
    },
    onComplete: (error) {
        if (error == GemError.success) {
            print("The update process finished successfully");
        } else {
            print("The update process failed! The error code is $error");
        }
    },
);
```

## Content updater status

The `ContentUpdaterStatus` enum provided by the `onStatusUpdated` method has the following values:

| Enum Value                  | Description |
|-----------------------------|-------------|
| `idle`                      | The update process has not started. It's the default state of the `ContentUpdater` |
| `waitConnection`            | Waiting for an internet connection to proceed (Wi-Fi or mobile). |
| `waitWIFIConnection`        | Waiting for a Wi-Fi connection before continuing. Available if the `update` method has been called with `false` value for the `allowChargeNetwork` parameter. |
| `checkForUpdate`            | Checking for available updates. |
| `download`                  | Downloading the updated content. The overall progress and the per-item progress is available. |
| `fullyReady`                | The update is fully downloaded and ready to be applied. The `ContentUpdater.items` list will contain the target items for the update. An `apply` call is required for the update to be applied|
| `partiallyReady`            | The update is partially downloaded but can still be applied. The content available offline that wasn't yet updated will be deleted if the update is applied and the remaining items will be updated. |
| `downloadRemainingContent`  | Downloading any remaining content after applying the update. If the `apply` method is called while the `partiallyReady` status is set, then this will be the new status. The user must get the list of remaining content items from the updater and start  normal download operation. |
| `downloadPendingContent`    | Downloads any pending content that has not yet been retrieved. If a new item starts downloading during an update, it will complete after the update finishes (at the latest version). This value is provided while these downloads are in progress.|
| `complete`                  | The update process has finished successfully. The `onComplete` callback is also triggered with `GemError.success` |
| `error`                     | The update process encountered an error. The `onComplete` callback is also triggered with the appropriate error code |

---

## Get ContentUpdater details

Get details about the `ContentUpdater` object using the provided getters:
```dart
final ContentUpdaterStatus status = contentUpdater.status;
final int progress = contentUpdater.progress;
final bool isStarted = contentUpdater.isStarted;
final bool canApplyUpdate =  contentUpdater.canApply;
final bool isUpdateStarted = contentUpdater.isStarted;
```

---

## Apply the update

Once `onStatusUpdated` is called with a `ContentUpdaterStatus` value of `fullyReady` or `partiallyReady`, apply the update using the `apply` method of the `ContentUpdater` class.

- If the status is `fullyReady`, all items have been downloaded

- If the status is `partiallyReady`, only a subset of the items has been downloaded. Applying the update will remove outdated items that were not fully downloaded, restricting offline functionality to the updated content only. The remaining items will continue downloading
```dart
onStatusUpdated: (ContentUpdaterStatus status) {
    print("OnStatusUpdated with code $status");
    if (status == ContentUpdaterStatus.fullyReady ||
        status == ContentUpdaterStatus.partiallyReady) {

        if (!contentUpdater.canApply){
            print("Cannot apply content update");
            return;
        }
        final applyError = contentUpdater.apply();
        print("Apply resolved with code ${applyError.code}");
    }
}
```

The `apply` method returns:

- `GemError.success` if the update was applied successfully

- `GemError.upToDate` if the update is already up to date and no changes were made

- `GemError.invalidated` if the update operation has not been started successfully via the `update` method

- `GemError.io` if an error regarding the file system occurred while updating the content

The `onComplete` callback is also triggered with the appropriate error code.

---

## Update resources

The Magic Lane Flutter SDK includes built-in resources such as icons and translations. Enable or disable automatic updates for these resources by setting the `isAutoUpdateForResourcesEnabled` field within the `AutoUpdateSettings` object passed to `GemKit.initialize`.

If you configure callbacks manually using the `setAllowConnection` method, resource updates can still be enabled by setting the optional `canDoAutoUpdateResources` parameter.

Unlike other content types, updating these resources does not require a `ContentUpdater`. By default, resource updates are enabled (`isAutoUpdateForResourcesEnabled` is true, and `canDoAutoUpdateResources` is true).

---

## Relevant examples demonstrating content update related features

- [Map Update](/examples/maps-3dscene/map-update)

- [Map Style Update](/examples/maps-3dscene/map-style-update)
