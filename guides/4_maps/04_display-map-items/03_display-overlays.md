---
description: Documentation for Display Overlays
title: Display Overlays
---

# Display overlays

Overlays are used to provide enhanced, layered information that adds contextual value to a base map, offering users dynamic, interactive, and customized data that can be visualized on top of other map elements.

### Disabling overlays

Overlays can be disabled either by applying a predefined, custom map style created in [Magic Lane Map Studio](https://developer.magiclane.com/documentation/OnlineStudio/guide_creating_a_style.html)—where certain overlays are already disabled—or by using the ``disableOverlay`` method, as shown below:
```dart
GemError error = OverlayService.disableOverlay(CommonOverlayId.publicTransport.id);
```

Passing -1 (default value) as the optional `categUid` parameter indicates that we want to disable the entire public transport overlay, rather than targeting a specific category.

The error returned will be `success` if the overlay was disabled or `notFound` if no overlay (or overlay category) with the specified ID was found in the applied style.

To disable specific overlays within a category, you'll need to retrieve their unique identifiers (uid) as shown below:
```dart 
final Completer<GemError> completer = Completer<GemError>();
final availableOverlays = OverlayService.getAvailableOverlays(onCompleteDownload: (error) {
  completer.complete(error);
});

await completer.future;
  
final collection = availableOverlays.first;
final overlayInfo = collection.getOverlayAt(0);
if(overlayInfo != null) {
  final uid = overlayInfo.uid;
  final err = OverlayService.disableOverlay(uid);
  showSnackbar(err.toString());
}
```

### Enabling overlays

In a similar way, the overlay can be enabled using the ``enableOverlay`` method by providing the overlay id. It also has an optional `categUid` parameter, which when left as default, it activates whole overlay rather than a specific category.

## Relevant examples demonstrating overlays related features

- [Create Custom Overlay](/examples/places-search/create-custom-overlay)
