---
description: Documentation for Roadmap And Known Issues
title: Roadmap And Known Issues
---

# Roadmap and known issues

The Maps SDK for Flutter is currently in a stable release. We are actively investing in further improvements, new features, and increased platform coverage. This document outlines the upcoming roadmap as well as known issues that are currently being addressed.

## Roadmap

The following features and improvements are planned for future releases of the Maps SDK for Flutter:

- **Embedded Linux Support**  

    Planned support for embedded Linux devices, expanding the range of platforms where the Maps SDK for Flutter can be utilized.

- **Web Support**  

    Upcoming support for web applications, enabling the development of cross-platform mapping solutions that run directly in modern browsers.

- **Performance Enhancements**  

    Ongoing performance optimizations to ensure smooth and responsive user experiences.

- **API Consistency Improvements**  

    Refinements across SDK modules to improve API consistency and usability, aligned with Flutter development best practices.

- **Documentation Enhancements**  

    Continuous improvements to documentation, including additional examples, guides, and a comprehensive list of possible errors with detailed explanations in method-level documentation.

- **Cross-Platform Consistency**  

    Improvements aimed at ensuring consistent behavior and feature parity across all supported platforms, providing a seamless development experience.

- **Example Projects: AGP Upgrade**  

    Updates to example projects to use the latest Android Gradle Plugin (AGP) versions, ensuring compatibility with current Android development standards.

- **Extended Support for Older Android Versions**  

    Exploration of approaches to broaden support for older Android versions while maintaining SDK performance and stability.

- **Expanded OSM Contact Data Support**  

    Enhanced support for OpenStreetMap (OSM) contact data to provide richer and more comprehensive map information.

- **Quality of Life Improvements**  

    Various usability enhancements and refinements to improve the overall developer experience, especially for common use cases.

- **New Features**  

  Active development of new capabilities, including:

  - EV (Electric Vehicle) routing

  - VRP (Vehicle Routing Problem) routing

  - Additional methods and enhancements to existing classes

## Known Issues

The following known issues are currently being addressed:

- **Stability on Certain Android Devices**  

    Stability issues have been observed on some Android devices, particularly those with lower-end hardware or with x32 bit architectures. These are under active investigation, and fixes are being prioritized to improve overall robustness.

- **Image UID Inconsistencies**  

    Some vector-based image objects may have different UIDs even when their visual content is identical.

- **Map Clipping on Android**  

    Occasional map clipping may occur on certain Android devices when adding UI elements on top of `GemMap` widgets. These issues are related to known bugs in the Flutter framework and are being closely monitored.  
    See: [Flutter issue #164899](https://github.com/flutter/flutter/issues/164899)

- **Platform-Specific Event Availability**  

    Certain `OffboardListener` and `NetworkProvider` callbacks are not supported on all platforms. Work is ongoing to improve event handling consistency.

- **Undefined Behavior for Invalid Parameters**  

    Some methods (such as the `cloneStartEnd` method of the `Path` class) may exhibit undefined behavior when provided with invalid parameters. Improved validation, error handling, and clearer documentation are planned to address this.

- **Texture View Mode Limitations**  

    Some operations are not supported when using texture view rendering mode. These limitations include the inability to properly release widgets and the inability to use texture view concurrently with other `GemMap` widgets that rely on different rendering modes, as gesture handling is not fully supported in such scenarios.
    Please keep in mind that the texture view mode is experimental and may change or be removed in future releases.

- **Crashes During Repeated Map Style Changes**  

    Changing the map style repeatedly in a short period may lead to application crashes. This issue is under investigation, and fixes are being implemented to enhance stability during style transitions.

- **Incorrect Map Highlighting on Certain Scenarios**  

    Some scenarios may lead to incorrect highlighting of map elements, such as routes or markers. Efforts are underway to identify the root causes and implement solutions to ensure accurate map rendering, acording to specified parameters.
