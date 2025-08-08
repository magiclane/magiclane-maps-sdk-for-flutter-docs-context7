---
description: Documentation for Update Content
title: Update Content
---

# Update Content

The Magic Lane SDK allows updating downloaded content to stay synchronized with the latest map data. New map versions are released every few weeks.
The update operation supports the `roadMap`, `viewStyleLowRes`, and `viewStyleHighRes` content types. This article focuses on the `roadMap` type, as it is the most common use case. Updating styles mostly follows the same process.

The SDK requires all road map content store items to have the same version. It is not possible to have multiple `roadMap` items with different versions, meaning partial updates of individual items are not supported.

Based on the client's content version relative to the newest available release, it can be in one of three states, as defined by the `Status` enum:

| Status        | Description  |
|---------------|--------------|
| `expiredData` | The client version is significantly outdated and no longer supports online operations.  All features—such as search, route calculation, and navigation—will function exclusively on the device using downloaded regions, even if an internet connection is available. If an operation like route calculation is attempted in non-downloaded regions, it will fail with a `GemError.expired` error code. An update is **mandatory** to restore online functionality. |
| `oldData`     | The client version is outdated but still supports online operations.  Features will work online when a connection is available and offline when not. While offline, only downloaded regions are accessible, but online access allows operations across the entire map.  An update is **recommended** as soon as possible. |
| `upToDate`    | The client version has the latest map data. Features function online when connected and offline using downloaded regions.  No updates are available. |

The Magic Lane servers support online operations for the most up-to-date version (`Status.upToDate`) and the previous version (`Status.oldData`).

The Magic Lane Flutter SDK is designed for seamless automatic updates by default, ensuring compatibility with the latest map data with minimal effort from the API user. If manual map update management is not required for your use case, no additional configuration is needed.

After installation, the app includes a default map version of type `expiredData`, which contains no available content. Therefore, an update—either manually triggered or performed automatically—is required before the app can be used.
Internet access is required for the initial use of the app.

## Update process overview

1. The map update process is initiated by the API user or the SDK automatically starts the download process.
2. The process downloads the newer data in background ensuring the full usability of the current ( old ) map dataset for browsing, search and navigation. The content is downloaded in a close-to-current user position order, i.e. nearby maps are downloaded first.
3. Once all new version data is downloaded:

    - If the auto-update feature is enabled, then the update is automatically applied

    - If the user initiated the update manually, the API user is notified and the update is applied by replacing the files is an atomic operation

If the user's storage size does not allow the existence of the old and new dataset at the same time, the update required an additional step:

4. The remaining offline maps which did not download because of the out-of-space exception should be downloaded by the API user by an usual call to the `ContentStoreItem.asyncDownload`.

The auto-update behavior is different between the Magic Lane SDKs:

- The C++ SDK does not provide an auto-update mechanism

- The Kotlin SDK provides an auto-update mechanism

- The iOS SDK does not provide an auto-update mechanism

## Listen for the auto-update completion

The `registerOnAutoUpdateComplete` method from the `OffBoardListener` class can be used to listen for auto-update completion events.
```dart
SdkSettings.offBoardListener.registerOnAutoUpdateComplete((ContentType type, GemError error) {
    if (error == GemError.success) {
        print("The update process finished successfully for $type");
    } else {
        print("The update process failed for $type! The error code is $error");
    }
});
```

The callback will be triggered for each type of content when the auto update process finishes (only for types configured to auto update).

If the auto-update fails then the API user is responsible to treat this case and trigger an update manually if needed.

## Disable automatic updates

By default, automatic updates are enabled for road maps and styles. You can configure this behavior using the `AutoUpdateSettings` class, which can be passed to the `GemKit.initialize` method at the start of the application.

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

The auto update settings can also be configured later in the application in the `OffBoardListener` class:
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

If the auto update settings have been changed via the `SdkSettings.offBoardListener` object then a call to the `SdkSettings.autoUpdate()` is required in order to check for new updates and automatically download and apply.

If the update has already been done for a specific type, the auto update configuration for that type is no longer taken into account. It is not possible to return to an older version if an update has been applied.

The `AutoUpdateSettings` class also includes the `AutoUpdateSettings.allDisabled()` and `AutoUpdateSettings.allEnabled()` constructors for quickly disabling or enabling all updates.

## Listen for map updates

You can listen for map updates by calling the `registerOnWorldwideRoadMapSupportStatus` method and providing a callback.
```dart
SdkSettings.offBoardListener.registerOnWorldwideRoadMapSupportStatus((status){
        switch (status) {
          case MapStatus.upToDate:
            showSnackbar("The map version is up-to-date.");
            break;
          case MapStatus.oldData:
            showSnackbar(
                "A new map version is available. Online operation on the current map version are still supported.");
            break;
          case MapStatus.expiredData:
            showSnackbar(
                "The map version has expired. All operations will be executed offline.");
            break;
        }
      },
    );
```

The SDK may automatically trigger the map version check at an appropriate moment. To manually force the check, you can call the `checkForUpdate` method from the `ContentStore`:
```dart
final GemError checkUpdateCode = ContentStore.checkForUpdate(ContentType.roadMap);
print('Check for update result code $checkUpdateCode');
```

The `checkForUpdate` method returns `GemError.success` if the check has been initiated and `GemError.connectionRequired` if no internet connection is available.

If the `checkForUpdate` method is provided with the `ContentType.roadMap` argument, then the `onWorldwideRoadMapSupportStatusCallback` will be called.
If other values are supplied to the `checkForUpdate` method (such as map styles), then the response will be returned via the `onAvailableContentUpdateCallback` callback.

## Create content updater

To update the road maps, you must instantiate a `ContentUpdater` object. This object manages all operations related to the update process:
```dart
final (contentUpdater, contentUpdaterCreationError) = ContentStore.createContentUpdater(ContentType.roadMap);

if (contentUpdaterCreationError != GemError.success && contentUpdaterCreationError != GemError.exist){
    print("Error regarding the content updater creation : $contentUpdaterCreationError");
    return;
}
```

The `createContentUpdater` method returns a `ContentUpdater` instance along with an error code indicating the status of the updater creation for the specified `ContentType`:

- If the error code is `GemError.success`, the `ContentUpdater` was successfully created and is ready for use.

- If the error code is `GemError.exist`, a `ContentUpdater` for the specified `ContentType` already exists, and the existing instance is returned. This instance remains valid and can be used.

- If the error code corresponds to any other `GemError` value, the `ContentUpdater` instantiation has failed, and the returned object is not usable.

## Start the update

In order to start the update process, you can call the `update` method from the `ContentUpdater` object created earlier.
The `update` method accepts the following parameters:

- a boolean indicating whether the update can proceed on networks that may incur additional charges. If false, the update will only run on free networks, such as Wi-Fi.

- a callback named `onStatusUpdated` that will be `triggered` when the update status changes, providing information through the `ContentUpdaterStatus` parameter.

- a callback named `onProgressUpdated` that will be triggered when the update progresses, supplying an integer value between 0 and 100 to indicate completion percentage.

- a callback named `onComplete` that will be triggered with the update result at the end of the update process (after calling `apply` or if the update fails earlier). The most common error codes are:

    - `GemError.success` if the update was successful.

    - `GemError.inUse` if the update is already running and could not be started.

    - `GemError.notSupported` if the update operation is not supported for the given content type

    - `GemError.noDiskSpace` if there is not enough space on the device for the update

    - `GemError.io` if an error regarding file operations occurred

The method returns a non-null `ProgressListener` if the update process has been successfully started or `null` if the update couldn't be started (case in which the `onComplete` will be called with the error code). 
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

## ContentStore details

Details about the `ContentUpdater` objects can be got via the provided getters:
```dart
final ContentUpdaterStatus status = contentUpdater.status;
final int progress = contentUpdater.progress;
final bool isStarted = contentUpdater.isStarted;
final bool canApplyUpdate =  contentUpdater.canApply;
final bool isUpdateStarted = contentUpdater.isStarted;
```

## Finish the update process

Once `onStatusUpdated` is called with a `ContentUpdaterStatus` value of `fullyReady` or `partiallyReady`, the update can be applied using the `apply` method of the `ContentUpdater` class.

- if the status is `fullyReady`, all items have been downloaded.

- if the status is `partiallyReady`, only a subset of the items has been downloaded. Applying the update will remove outdated items that were not fully downloaded, restricting offline functionality to the updated content only. The remaining items will continue downloading.
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

- `GemError.success` if the update was applied successfully.

- `GemError.upToDate` if the update is already up to date and no changes were made.

- `GemError.invalidated` if the update operation has not been started successfully via the `update` method.

- `GemError.io` if an error regarding the file system occurred while updating the content.

The `onComplete` will also be triggered with the appropriate error code.

## Update resources

The Magic Lane Flutter SDK includes built-in resources such as icons and translations. Automatic updates for these resources can be enabled or disabled by setting the `isAutoUpdateForResourcesEnabled` field within the `AutoUpdateSettings` object passed to `GemKit.initialize`.

If the API user configures callbacks manually using the `setAllowConnection` method, resource updates can still be enabled by setting the optional `canDoAutoUpdateResources` parameter.

Unlike other content types, updating these resources does not require a `ContentUpdater`, simplifying the process. By default, resource updates are enabled (`isAutoUpdateForResourcesEnabled` is true, and `canDoAutoUpdateResources` is true).
