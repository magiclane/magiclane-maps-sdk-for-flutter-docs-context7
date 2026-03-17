---
description: Documentation for Migrate To 3 1 8
title: Migrate To 3 1 8
---

# Migrate to 3.1.8

This guide outlines the breaking changes introduced in SDK version 3.1.8. Required updates may vary depending on your use case.

Additionally, new features and bugfixes have been introduced and are **not** documented here. For a comprehensive list of changes, please refer to the changelog.

This release focuses on fixes and deprecations that will allow for better performance when accessing some items.

- `getRoadInfoImg` method from the `NavigationInstruction` class as it was deprecated. Use the `getRoadInfoImgByType` method instead.

## The **getRoadInfoImg** method from the **NavigationInstruction** class has been deprecated

The newly introduced `getRoadInfoImgByType` method will allow removing the need for `RoadInfo` to be dealocated when removed by the garbage collector, which will provide significant performance improvements as many `RoadInfo` objects instances are created when receiving improved position updates.

Before:
```dart
RoadInfoImg image = instruction.getRoadInfoImg(instruction.currentRoadInformation);
```

After:
```dart
RoadInfoImg image = instruction.getRoadInfoImgByType(RoadInfoType.currentRoadInformation);
```

Use the following `RoadInfoType` values to retrieve the corresponding `RoadInfoImg`:

- `currentRoadInformation` for `currentRoadInformation`

- `nextRoadInformation` for `nextRoadInformation`

- `nextNextRoadInformation` for `nextNextRoadInformation`

Please let us know if you had specific use cases for the `getRoadInfoImg` method that are not covered by the new `getRoadInfoImgByType` method so these can be accommodated before removing the deprecated method in a future release.
