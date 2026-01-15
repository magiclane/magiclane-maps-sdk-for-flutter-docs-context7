---
description: Documentation for Introduction
title: Introduction
---

# Introduction

The Maps SDK for Flutter provides offline functionality through map download capabilities.

Download maps for entire countries or specific regions to enable offline access. Users can search for landmarks, calculate routes, navigate, and explore maps without an internet connection.

Overlays, live traffic information, and other online-dependent services are unavailable in offline mode.

The SDK updates downloaded maps automatically by default. New map versions are released every few weeks globally, providing regular enhancements and improvements.

You can configure update preferences to restrict downloads to Wi-Fi connections only or permit cellular data usage. The SDK can also notify users about available updates.

## Offline feature availability

### Core entities

| Entity       | Offline availability                                                                                                           |
|--------------|--------------------------------------------------------------------------------------------------------------------------------|
| [Base entities](../core/base-entities) | ✓ Fully operational                      |
| [Position](../core/positions)          | ⚠️ Partial - Raw position data is always accessible, but map-matched position data requires downloaded or cached regions |
| [Landmarks](../core/landmarks)         | ✓ Fully available                                                                                    |
| [Markers](../core/markers)             | ✓ Fully available                                                                                     |
| [Overlays](../core/overlays)           | ✗ Not available                                                                                       |
| [Routes](../core/routes)               | ⚠️ Partial - The `trafficEvents` getter returns an empty list without internet connection       |
| [Navigation](../core/navigation-instructions) | ✓ Fully available if navigation starts on an offline-calculated route                             |

Map tiles are automatically cached based on your location, camera position, and calculated routes to enhance performance and offline accessibility.

### Map controller

The `GemMapController` methods function as expected in offline mode. Methods that request data from regions not covered or cached return empty results.

**Example:** Calling `GemMapController.getNearestLocations()` with `Coordinates` outside covered areas returns an empty list.

### Map styling

You can set a new map style in offline mode if the style has been downloaded beforehand.

Styles containing extensive data, such as Satellite and Weather styles, may not display meaningful information when offline.

### Services

The following services are available offline within downloaded map regions:

- `RoutingService`

- `SearchService`

- `GuidedAddressSearchService`

- `NavigationService`

- `LandmarkStoreService`

- `PositionService`

The following services are **not supported** in offline mode:

- `OverlayService`

- `ContentStore`

**Error handling:**

- `SearchService.search` with queries outside downloaded regions returns `GemError.notFound`

- `RoutingService.calculateRoute` outside downloaded regions returns `GemError.activation`

- Other services may return `GemError.connectionRequired`

`AlarmService` has limited functionality during offline sessions, as overlay-related features are unavailable.

### SDK settings

Most fields and methods in `SdkSettings` function independently of internet connection status, except authorization-related functionalities.

Authorization with `GEM_TOKEN` requires an active internet connection. You cannot authorize the Maps SDK for Flutter without being online. Invoking `SdkSettings.verifyAppAuthorization` returns `GemError.connectionRequired` if there is no internet connection.

## Relevant examples demonstrating offline related features

- [Offline Routing](/examples/routing-navigation/offline-routing)

- [Map Download](/examples/maps-3dscene/map-download)

- [Map Update](/examples/maps-3dscene/map-update)
