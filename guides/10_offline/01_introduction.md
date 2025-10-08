---
description: Documentation for Introduction
title: Introduction
---

# Introduction

The Maps SDK for Flutter provides extensive offline functionality through its map download capabilities. Users can search for landmarks, calculate and navigate routes, and explore the map without requiring an active internet connection.

However, certain features, such as overlays, live traffic information and other online-dependent services, are unavailable in offline mode.

The SDK empowers users to download maps for entire countries or specific regions directly to their devices, enabling seamless offline access to essential features. Additionally, it allows users to update their downloaded maps, ensuring they always have access to the latest data and improvements for offline use.

The SDK is designed to update downloaded maps automatically by default, ensuring users always have access to the latest data. New map versions are typically released every few weeks globally, providing regular enhancements and improvements.

Additionally, the SDK allows for user notifications about available updates, keeping them informed and in control. Developers can configure update preferences, such as restricting updates or downloads to Wi-Fi connections only, or permitting them over cellular data, offering flexibility based on user needs and network conditions.

## SDK Features available offline

### Core Entities

| Entity       | Offline Availability                                                                                                           |
|--------------|--------------------------------------------------------------------------------------------------------------------------------|
| [Base entities](../core/base-entities) | Fully operational in offline use cases, as they do not require an active internet connection.                      |
| [Position](../core/positions)          | Partially available. Raw position data is always accessible, but map-matched position data is only available if the relevant region has been previously downloaded or cached. |
| [Landmarks](../core/landmarks)         | Fully available in offline mode.                                                                                     |
| [Markers](../core/markers)             | Fully available in offline mode.                                                                                     |
| [Overlays](../core/overlays)           | Not available in offline mode.                                                                                       |
| [Routes](../core/routes)               | Partially available. The `trafficEvents` getter will return an empty list when there is no internet connection.       |
| [Navigation](../core/navigation-instructions) | Fully available in offline mode if navigation is started on an offline-calculated route.                              |

Map tiles are automatically cached based on the user's location, camera position, and calculated routes to enhance performance and offline accessibility.

### Map Controller

The `GemMapController` methods function as expected in offline mode. However, methods that request data from regions that are not covered or cached will return will return empty results (e.g., an empty list or an empty string, depending on the method). For instance, calling `GemMapController.getNearestLocations()` with `Coordinates` outside of covered areas will result in an empty list.

### Map Styling

Setting a new map style is supported in offline mode, provided the style has been downloaded beforehand. Note that styles containing extensive data, such as Satellite and Weather styles, may not display meaningful information when offline.

### Services

The following services are available offline only within downloaded map regions:

- `RoutingService`

- `SearchService`

- `GuidedAddressSearchService`

- `NavigationService`

- `LandmarkStoreService`

- `PositionService`

The remaining services such as `OverlayService` and `ContentStore` are **not supported** in offline mode.

Calling `SearchService.search` with queries outside of downloaded map regions will result in an `GemError.notFound`. Same goes for `RoutingService.calculateRoute`, which will return `GemError.activation`. Other services might return `GemError.connectionRequired`.

`AlarmService` has limited functionality during offline sessions, as overlay-related features are unavailable.

### Sdk Settings

Most fields and methods in `SdkSettings` function independently of the internet connection status, with the exception of authorization-related functionalities. The authorization with `GEM_TOKEN` **requires** an active internet connection. Consequently, you cannot authorize the Maps SDK for Flutter without being online. Invoking `SdkSettings.verifyAppAuthorization` will result in a `GemError.connectionRequired` error through its callback if there is no internet connection.

You can also retrieve certain SDK-related images in offline mode using the `SdkSettings.getImageById()` method, which accesses images stored in the `EngineMisc` enum.

## Relevant examples demonstrating offline related features

- [Offline Routing](/examples/routing-navigation/offline-routing)

- [Map Download](/examples/maps-3dscene/map-download)

- [Map Update](/examples/maps-3dscene/map-update)
