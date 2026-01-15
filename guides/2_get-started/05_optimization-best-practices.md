---
description: Documentation for Optimization Best Practices
title: Optimization Best Practices
---

# Optimization best practices

Optimize app performance and resource management with the Magic Lane SDK for Flutter. Follow these strategies to create smoother user experiences, reduce unnecessary processing, and maintain efficient use of memory and system resources.

---

## Minimize SDK method calls

Frequent calls to SDK methods can be computationally expensive and lead to UI lag, especially in performance-critical paths like rendering or user interactions.

### Cache static values

Retrieve values like IDs, names, or immutable properties once and store them locally. Identify which values can be cached and which need to be queried each time based on your use case.

Avoid retrieving large numbers of elements at once, as this increases memory usage and slows performance. Fetch values lazily when needed for the first time and cache them for future use.

### Avoid repeated collection queries

When accessing a collection multiple times within a short scope, store the result in a temporary variable rather than calling the SDK method repeatedly.

### Throttle or debounce rapid calls

Debounce SDK calls triggered by rapid user interactions (text fields, buttons, sliders) to ensure the method is invoked only once after a defined period of inactivity. This reduces redundant calls and improves responsiveness.

### Use listeners to detect changes

Register listeners provided by the SDK instead of polling to detect state changes. This allows your code to react only when changes occur and avoids unnecessary calls to check the SDK state.

### Use lazy initialization

Initialize or load SDK objects only when needed. Avoid early or unnecessary SDK calls during app startup or in unused paths. This improves app startup time.

### Avoid SDK calls in widget build methods

The `build` method is called multiple times, triggering unnecessary SDK calls each time the widget rebuilds. Compute values once and store them in state before the render cycle.

---

## Optimize image requests

The `Img`, `AbstractGeometryImg`, `LaneImg`, `SignpostImg`, and `RoadInfoImg` classes expose a `uid` getter that uniquely identifies each image instance.

During navigation, sequential `NavigationInstruction` objects may reference identical images. Avoid re-requesting or redrawing images if their `uid` matches the previous instruction.

---

## Avoid SDK calls during animations

Avoid SDK calls while UI animations are in progress. These calls can introduce delays or stuttering.

Schedule SDK calls to occur **before** the animation starts or **after** it completes. This maintains fluid animations and reduces dropped frames.

---

## Stop map rendering when not visible

Use the `isRenderEnabled` setter of the `GemMapController` class to control map rendering:

- Set to `false` to stop rendering when the map is not visible

- Set to `true` to enable rendering when the map becomes visible

Further improvements are needed for this functionality, especially on Android platforms.

---

## Release objects

Enable the garbage collector to reclaim resources tied to large objects once they're no longer needed. For immediate control over resource management, manually free resources by calling the object's `dispose` method.

Avoid storing large collections of entities in memory at one time.

Calling methods on disposed objects is unsupported and can lead to application crashes.

---

## Avoid unnecessary `GemMap` widgets

While multiple `GemMap` widgets can be used simultaneously, having many active at once negatively impacts performance.

Avoid maintaining lists of `GemMap` instances. Reuse a single map widget whenever possible rather than creating new ones on demand.

---

## Test Skia vs Impeller performance

In Flutter, Impeller generally offers better performance than Skia, especially on iOS, by reducing jank and improving animation smoothness for high-end devices.

However, Skia may outperform Impeller on some devices and use cases, particularly on Android or older devices. Performance varies depending on device GPU, driver support, and rendering patterns.
