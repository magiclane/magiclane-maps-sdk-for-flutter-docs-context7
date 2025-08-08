---
description: Documentation for Display Paths
title: Display Paths
---

# Display paths

[Paths](/guides/core/base-entities#path) can be displayed by adding them into ``MapViewPathCollection``. The ``MapViewPathCollection`` is an iterable collection, having fields like ``size``, ``add``, ``remove``, ``removeAt``, ``getPathAt`` and ``getPathByName``.
```dart
mapController.preferences.paths.add(path);
```

The ``add`` method of ``MapViewPathCollection`` includes optional parameters for customizing the appearance of paths on the map, such as ``colorBorder``, ``colorInner``, ``szBorder``, and ``szInner``. To center the map on a path, use the ``GemMapController.centerOnArea()`` method with the path's area retrieved from the area getter.
```dart
mapController.preferences.paths.add(path);

mapController.centerOnArea(path.area);
```

To remove all paths from ``GemMap`` use ``MapViewPathCollection.clear()``. To remove selectively, use ``MapViewPathCollection.remove(path)`` or ``MapViewPathCollection.removeAt(index)``.
