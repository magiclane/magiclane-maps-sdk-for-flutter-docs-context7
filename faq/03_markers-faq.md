---
description: Documentation for Markers Faq
title: Markers Faq
---

# Markers FAQ

<sub style={{ color: 'gray', display: 'inline-block' }}>
  
</sub>

# Why are my markers and groups labels not visible?

The labels won't be visible unless you specify the following values for labeling mode: `MarkerLabelingMode.itemLabelVisible`, `MarkerLabelingMode.groupLabelVisible`.
```dart
final renderSettings = MarkerCollectionRenderSettings(labelingMode: {
            //highlight-start
            MarkerLabelingMode.itemLabelVisible,
            MarkerLabelingMode.groupLabelVisible,
            //highlight-end
      },
      image: GemImage(image: image, format: ImageFileFormat.png)
);

_controller.preferences.markers.add(pointMarkerCollection, settings: renderSettings);
```


