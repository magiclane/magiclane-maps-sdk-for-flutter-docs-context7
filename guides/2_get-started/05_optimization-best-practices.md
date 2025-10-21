---
description: Documentation for Optimization Best Practices
title: Optimization Best Practices
---

# Optimization best practices

This guide highlights key strategies to improve app performance and resource management when using the Magic Lane SDK for Flutter.

By following these best practices, you can create smoother user experiences, reduce unnecessary processing, and maintain efficient use of memory and system resources.

## Minimize SDK method calls

Frequent calls to SDK methods, including getters, setters, and object constructors, can be computationally expensive and may lead to UI lag or degraded performance, especially in performance-critical paths such as rendering or user interactions.

In order to improve efficiency and avoid potential performance issues:

#### Cache static or infrequently changing values

For example, values like IDs, names, or other immutable properties should be retrieved once and stored locally. Repeatedly querying such values from the SDK can introduce unnecessary overhead. Identify which values can be cached and which values are often changing and need to be queried each time depending on your usecase.

Avoid retrieving a large number of elements all at once, as this can lead to increased memory usage and slower performance. 

A more efficient approach may be, depending on your usecase, to fetch values lazily—only when they are needed for the first time—and cache them for future use. This ensures optimal resource utilization while maintaining responsiveness.

#### Avoid repeated collection queries

When accessing a collection (such as lists of elements) from the SDK multiple times within a short scope, store the result in a temporary variable rather than calling the SDK method repeatedly.

#### Throttle or debounce rapid calls

Debouncing ensures that a function is executed only once after a specified delay, and only after the final event in a rapid sequence of interactions. This pattern is particularly useful in scenarios involving user interaction with elements such as text fields, buttons, sliders, or other dynamic UI components.

When such interactions trigger SDK method calls in quick succession, it can lead to performance issues or unnecessary processing. Instead, debounce these calls to ensure the SDK method is invoked only once, after a defined period of inactivity. This approach reduces redundant calls and improves overall application responsiveness.

#### Use the provided listeners to detect changes

Instead of polling the SDK to detect state changes (which generates repeated calls), register listeners provided by the SDK. This allows your code to react to changes only when they occur and avoids unnecessary calls to check the SDK state.

#### Lazy initialization and deferred loading

Initialize or load SDK objects when they are actually needed. Avoid early or unnecessary SDK calls during app startup or in unused paths.
This approach can greatly improve the performance of the app startup time.

#### Avoid calling SDK methods inside the build methods of the widgets

As the `build` method of the widgets are called multiple times, unnecessary calls to the SDK may be done each time the widget rebuilds. Instead, compute the values once and store them in state before the render cycle.

## Request and redraw images only when the image id changes

The `Img`, `AbstractGeometryImg`, `LaneImg`, `SignpostImg`, and `RoadInfoImg` classes expose a `uid` getter that uniquely identifies each image instance.

During navigation, it's possible for sequential `NavigationInstruction` objects to reference identical images. To optimize performance, avoid re-requesting or redrawing images if their `uid` matches those from the previous instruction.

## Avoid calls to the SDK when UI animations are ongoing

To ensure smooth user experience and prevent potential performance issues, avoid making calls to the SDK while UI animations are in progress. These calls can introduce delays or cause stuttering in the animation rendering.

Whenever possible, schedule SDK calls to occur either **before** the animation starts or **after** it has fully completed. This approach helps maintain fluid animations and reduces the risk of dropped frames or UI lag.

## Stop the rendering of the map when the widget is not visible

Use the `isRenderEnabled` setter of the `GemMapController` class to disable or enable the rendering of the map:

- set the rendering to false to stop the map rendering when the map is not visible.

- set the rendering to true to enable the map rendering when the map is again visible.

Further improvements are needed regarding the functionality, especially on Android platforms.

## Release objects

Enable the garbage collector to reclaim resources tied to large objects once they are no longer needed. 
For more immediate or fine-grained control over resource management, you can manually free resources by calling the object's `dispose` method.

Avoid storing large collections of entities in the memory at one time.

Calling methods on disposed objects is unsupported and can lead to application crashes. 

## Avoid the unnecessary creation of `GemMap` widgets

While multiple `GemMap` widgets can be used simultaneously, having a large number of them active at once may negatively impact performance.

To optimize, avoid maintaining a list of `GemMap` instances.
Also reuse a single map widget whenever possible rather than creating new ones on demand.

## Check if performance using Skia is better than Impeller

In Flutter, Impeller generally aims for better performance than Skia, especially on iOS, by reducing jank and improving animation smoothness, particularly for high-end devices and performance-critical rendering.

However, on some devices and specific use cases, Skia may still outperform Impeller, particularly on Android or when supporting a wider range of older devices. This can vary depending on the device GPU, driver support, and specific rendering patterns used in the app.
