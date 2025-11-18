---
description: Documentation for Camera Feed
title: Camera Feed
---

# Camera Feed

In addition to sensor data, the SDK `DataSource` provides access to recorded and live frames from the device camera. This is useful for both live camera feeds and recorded logs.

Recorded logs with camera data are MP4 files that can be played back using standard video players. Using external players such as [`video_player`](https://pub.dev/packages/video_player) is the preferred approach for most log-playback scenarios, as they can provide better performance.

## Camera Player Widget

The `GemCameraPlayer` widget can be used to display the camera feed on screen. Wrap the widget in a `SizedBox` or similar to control its size.
The `GemCameraPlayer` takes the following parameters:

- `controller` — the `GemCameraPlayerController` that drives playback and provides camera frames.

    Required. Connects the widget to the source of frames, exposes playback status
    (loading, playing, error, ended), and is used internally to start/stop
    rendering and request frame updates. See more information below.

- `fallbackWidget` — optional widget shown when a frame cannot be decoded or an

    error state is reported by the controller. Useful for showing an error placeholder or retry affordance.

- `loadingWidget` — optional widget shown while the player is waiting for the

    first frame or otherwise in a loading state. Commonly a spinner or shimmer to indicate work in progress.

- `endWidget` — optional widget shown once the camera feed or recording has

    finished and the controller reports an ended state. Useful for displaying a replay button or end-of-stream message.

- `fit` — optional `BoxFit` describing how the decoded image should be sized into

    the available space. Defaults to a cover-like behavior when not supplied. See the [BoxFit API reference](https://api.flutter.dev/flutter/painting/BoxFit.html)
    for more information.

## Camera Player Controller

The `GemCameraPlayerController` is the main interface for controlling camera feed playback and accessing frames.
It is created via the constructor, which takes a `DataSource` as a required parameter. Optionally, you can provide a `CameraConfiguration` to the `configurationOverride` parameter
to customize the frame size.

The `DataSource` provided to the `GemCameraPlayerController` must have the `DataType.camera` data type enabled.
Providing a `DataSource` without camera data will result in an error state in the controller and cause the `fallbackWidget` to be shown in the `GemCameraPlayer`.

When presenting the **live** camera feed, a **video** recording session must be active on the `DataSource`. See the [Video Recording guide](./recorder#record-video) for
more information on starting a video recording session.

### Camera Player Controller Properties

The `GemCameraPlayerController` exposes the following properties:

- `datasource`: the `DataSource` provided during construction. It can be used to query additional information about the source of camera frames and to control the playback session.

- `size`: the size of the camera frames being provided by the controller, taking into account any configuration overrides.

- `status`: the current playback status of the controller. The following statuses are possible:

    - `loading`: the controller is waiting for the first frame to be available. If the controller is used within a `GemCameraPlayer`, the `loadingWidget` will be shown during this state.

    - `error`: an error occurred while trying to provide frames (for example, the data source does not provide camera data or the raw camera frame data is in an unsupported format). If the controller is used within a `GemCameraPlayer`, the `fallbackWidget` will be shown during this state.

    - `playing`: the controller is actively providing frames. If the controller is used within a `GemCameraPlayer`, the camera feed will be rendered on screen during this state.

    - `paused`: the controller has paused frame delivery. If the controller is used within a `GemCameraPlayer`, the last rendered frame will remain on screen during this state.

    - `ended`: the camera feed or recording has finished. If the controller is used within a `GemCameraPlayer`, the `endWidget` will be shown during this state.

- `camera`: the `Camera` instance providing camera parameters such as intrinsics and extrinsics. It will be `null` if no camera data is available from the `DataSource`.

### Pause and Resume Camera Feed

The `GemCameraPlayerController` provides methods to pause and resume the camera feed:

- `pause()`: Pauses the camera feed. The controller's status will change to `paused`.

- `resume()`: Resumes the camera feed if it was paused. The controller's status will change back to `playing`.

Pausing and resuming the camera feed does not affect the underlying `DataSource` playback or recording session; it only controls the delivery of camera frames to the controller.
Use the `Playback` API available on the `DataSource` to control overall playback or recording for log data sources.

### Listening to Camera Player Controller Changes

The `GemCameraPlayerController` extends `ValueNotifier`, so you can listen to it for changes in playback status and update your UI accordingly.
The `GemCameraPlayerValue` provides access to the data source, the current playback status, the current `Camera` (if available), and the data source listener used internally to receive camera frames.
With this, you can listen for changes in the controller's value and react to status updates or errors.

### Lifecycle Management

Remember to dispose of the `GemCameraPlayerController` when it is no longer needed to free up resources.
Disposing the controller will stop the internal data source listener and release any associated resources.
The `GemCameraPlayer` widget does not automatically dispose of the controller, so ensure you manage its lifecycle appropriately in your application.
Dispose of both the controller and the player widget when appropriate, as they can consume significant resources.

## Relevant example demonstrating camera feed related features

- [Camera Feed](/examples/routing-navigation/camera-feed)
