---
description: Documentation for Show Your Location On The Map
title: Show Your Location On The Map
---

# Show location on map

The location of the device is shown by default using an arrow position tracker. If ``setLiveDataSource`` has been successfully set and the required permissions were granted then the position tracker showing the current location should be visible on the map as an arrow.

At the moment it is not possible to have multiple position trackers on the map.

GPS accuracy may be limited in environments such as indoor spaces, areas with weak GPS signals, or locations with significant obstructions, such as narrow streets or between tall buildings. In these situations, the tracker may exhibit erratic movement within a confined area. Additionally, the performance of device sensors, such as the accelerometer and gyroscope, can further impact GPS positioning accuracy.

This behavior is more pronounced when the device is stationary.

## Start follow position

Following the position tracker can be done calling the `startFollowingPosition` method on the mapController.
```dart
mapController.startFollowingPosition();
```

When the ``startFollowingPosition`` method is called, the camera enters a mode where it automatically follows the movement and rotation of the position tracker. This ensures the user's current location and orientation are consistently centered and updated on the map.

The ``startFollowingPosition`` method can take parameters such as ``animation``, which controls the movement from the current map camera position to the position of the tracker and ``zoomLevel`` and ``viewAngle``.

### Set map rotation mode

When following position, it is possible to have the map rotated with the user orientation:
```dart
final prefs = mapController.preferences.followPositionPreferences;
prefs.setMapRotationMode(FollowPositionMapRotationMode.positionHeading);
```

If you want to use the compass sensor for map rotation use `FollowPositionMapRotationMode.compass`:
```dart
prefs.setMapRotationMode(FollowPositionMapRotationMode.compass);
```

The map rotation can also be fixed to a given angle using the `FollowPositionMapRotationMode.compass` value and providing a `mapAngle` value:
```dart
prefs.setMapRotationMode(FollowPositionMapRotationMode.fixed, mapAngle: 30);
```

A value of `0` given for the `mapAngle` parameter represents north-up alignment

The `mapRotationMode` returns a record containing:

- the current `FollowPositionMapRotationMode` mode

- the map angle set in case of `FollowPositionMapRotationMode.fixed`

## Exit follow position 

The ``stopFollowingPosition`` method from the mapController can be used to programmatically stop following the position.
```dart
mapController.stopFollowingPosition();
```

The follow mode will be exited automatically if the user interacts with the map. Actions such as panning, or tilting will disable the camera's automatic tracking. This can be deactivated by setting ``touchHandlerExitAllow`` to false (see the section below).

## Customize follow position settings

The ``FollowPositionPreferences`` class has options which can be used to customize the behavior of following the position. This can be accessed from the ``preferences`` getter of the mapController.

The fields defined in `FollowPositionPreferences` take effect only when the camera is in follow position mode. To customize camera behavior when not following the position, refer to the fields available in `MapViewPreferences` and `GemMapController`.

<table>
  <tr>
    <th>Field</th>
    <th>Type</th>
    <th>Explanation</th>
  </tr>
  <tr>
    <td>cameraFocus</td>
    <td>Point</td>
    <td>The position on the viewport where the position tracker is located on the screen.</td>
  </tr>
  <tr>
    <td>timeBeforeTurnPresentation</td>
    <td>int</td>
    <td>The time interval before starting a turn presentation</td>
  </tr>
  <tr>
    <td>touchHandlerExitAllow</td>
    <td>bool</td>
    <td>If set to false then gestures made by the user will exit follow position mode</td>
  </tr>
  <tr>
    <td>touchHandlerModifyPersistent</td>
    <td>bool</td>
    <td>If set to true then changes made by the user using gestures are persistent</td>
  </tr>
  <tr>
    <td>viewAngle</td>
    <td>double</td>
    <td>The viewAngle used within follow position mode</td>
  </tr>
  <tr>
    <td>zoomLevel</td>
    <td>int</td>
    <td>The zoomLevel used within follow position mode</td>
  </tr>
  <tr>
    <td>accuracyCircleVisibility</td>
    <td>bool</td>
    <td>Specifies if the accuracy circle should be visible (regardless if is in follow position mode or not)</td>
  </tr>
  <tr>
    <td>isTrackObjectFollowingMapRotation</td>
    <td>bool</td>
    <td>Specifies if the track object should follow the map rotation</td>
  </tr>
</table>

Please refer to the [adjust map guide](../maps/adjust-map) for more information about the ``viewAngle``, ``zoomLevel`` and ``cameraFocus`` fields.

If no zoom level is set, a default value is used.

#### Use of *touchHandlerModifyPersistent*

When the camera enters follow position mode and manually adjusts the zoom level or view angle, these modifications are retained until the mode is exited, either manually or programmatically.

If `touchHandlerModifyPersistent` is set to `true`, then invoking `startFollowingPosition` (with default parameters for zoom and angle) will restore the zoom level and view angle from the previous follow position session.

If `touchHandlerModifyPersistent` is set to `false`, then calling `startFollowingPosition` (with default zoom and angle parameters) will result in appropriate values for the zoom level and view angle being recalculated.

It is recommended to set the `touchHandlerModifyPersistent` property value right before calling the `startFollowingPosition` method.

#### Use of *touchHandlerExitAllow*

If the camera is in follow position mode and the `touchHandlerExitAllow` property is set to `true`, a two-finger pan gesture in a non-vertical direction will cause the camera to exit follow position mode.

If `touchHandlerExitAllow` is set to false, the user cannot manually exit follow position mode through touch gestures. In this case, the mode can only be exited programmatically by calling the `stopFollowingPosition` method.

### Set circle visibility

For example, in order to show the accuracy circle visibility on the map (which is by default hidden):
```dart
FollowPositionPreferences prefs = mapController.preferences.followPositionPreferences;
GemError error = prefs.setAccuracyCircleVisibility(true);
```

### Customize circle color

The accuracy circle color can be set using the `setDefPositionTrackerAccuracyCircleColor` static method from the `MapSceneObject` class: 
```dart
final GemError setErrorCode = MapSceneObject.setDefPositionTrackerAccuracyCircleColor(Colors.red);
print("Error code for setting the circle color: $setErrorCode");
```

It is recommended to use colors with partial opacity instead of fully opaque colors for improved visibility and usability.

The current color can be retrieved using the `getDefPositionTrackerAccuracyCircleColor` static method:
```dart
final Color color = MapSceneObject.getDefPositionTrackerAccuracyCircleColor();
```

The color can be reset to the default value using the `resetDefPositionTrackerAccuracyCircleColor` static method:
```dart
final GemError resetErrorCode = MapSceneObject.resetDefPositionTrackerAccuracyCircleColor();
print("Error code for resetting the circle color: $resetErrorCode");
```

### Set position of the position tracker on the viewport

In order to set the position tracker on a particular spot of the viewport while in follow position mode the ``cameraFocus`` property can be used:
```dart
// Calculate the position relative to the viewport
double twoThirdsX = 2 / 3;
double threeFifthsY = 3 / 5;
Point<double> position = Point(twoThirdsX, threeFifthsY);

// Set the position of the position tracker in the viewport
// while in follow position mode
FollowPositionPreferences prefs =
    mapController.preferences.followPositionPreferences;
GemError error = prefs.setCameraFocus(position);

mapController.startFollowingPosition();
```

The ``setCameraFocus`` method uses a coordinate system relative to the viewport, not physical pixels.
In this way ``Point(0.0, 0.0)`` corresponds with top left corner and ``Point(1.0, 1.0)`` corresponds with right bottom corner.  

## Customize position icon

The SDK supports customizing the position tracker to suit your application's requirements. For example, you can set a simple PNG as the position tracker using the following approach:
```dart
    // Read the file and load the image as a binary resource
    final imageByteData = (await rootBundle.load('assets/navArrow.png'));

    // Convert the binary data to Uint8List
    final imageUint8List = imageByteData.buffer.asUint8List();
    
    // Customize the position tracker
    MapSceneObject.customizeDefPositionTracker(imageUint8List, SceneObjectFileFormat.tex);
```

Besides simple 2D icons, 3D objects as `glb` files can be set. The format parameter of the customizeDefPositionTracker should be set to `SceneObjectFileFormat.tex` in this case.

At this moment it is not possible to set different icons for different maps.

Make sure the resource (in this example ``navArrow.png``) is correctly registered within the `pubspec.yaml` file. See the [Flutter documentation](https://docs.flutter.dev/ui/assets/assets-and-images) for more information.

## Other position tracker settings

Other settings such as scale and the visibility of the position tracker can be changed using the methods available on the ``MapSceneObject`` which can be obtained using ``MapSceneObject.getDefPositionTracker``.

### Change the position tracker scale

To change the scale of the position tracker we can use the ``scale`` setter:
```dart
// Get the position tracker
MapSceneObject mapSceneObject = MapSceneObject.getDefPositionTracker();

// Change the scale 
mapSceneObject.scale = 0.5;
```

A value of 1 corresponds with the default scale value. The parameter passed to the setter should be in the range ``(0, mapSceneObject.maxScale]``. The code snippet from above sets half the scale.

The scale of the position tracker stays constant on the viewport regardless of the map zoom level.

### Change the position tracker visibility

To change the visibility of the position tracker we can use the ``visibility`` setter:
```dart
// Get the position tracker
MapSceneObject mapSceneObject = MapSceneObject.getDefPositionTracker();

// Change the visibility
mapSceneObject.visibility = false;
```

The snippet above makes the position tracker invisible.
