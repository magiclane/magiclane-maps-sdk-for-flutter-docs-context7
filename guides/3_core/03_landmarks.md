---
description: Documentation for Landmarks
title: Landmarks
---

# Landmarks

A **landmark** is a predefined, permanent location that holds detailed information such as its name, address, description, geographic area, categories (e.g., Gas Station, Shopping), entrance locations, contact details, and sometimes associated multimedia (e.g., icons or images). It represents significant, categorized locations with rich metadata, providing structured context about a place.

## Landmark Structure

### Geographic Details

A landmark's position is defined by `coordinates` (centroid) and `geographicArea` (full boundary). The geographic area can be a circle, rectangle, or polygon. For specific bounding areas, use the `getContourGeographicArea` method.

Calculate the distance between two landmarks using the `distance` method:
```dart
final double distanceInMeters = landmark1.coordinates.distance(landmark2.coordinates);
```

See the [Coordinates](./base-entities) guide for more details.

### Waypoint Track Data

Some landmarks include a `trackData` attribute representing a sequence of waypoints that outline a path.

Available operations:

- `hasTrackData` - Returns `true` if the landmark contains track data

- `trackData` (getter) - Returns the track as a `Path` object (empty when no track exists)

- `trackData` (setter) - Replaces the landmark's track with a provided `Path`

- `reverseTrackData()` - Reverses the waypoint sequence

Waypoint track data is used for path-based routes. See [Compute path based route](/guides/routing/advanced-features) for details.

### Descriptive Information

Landmarks include `name`, `description`, and `author` attributes. Names adapt to SDK language settings for localization.

### Categories and Metadata

Landmarks can belong to one or more `categories` (described by `LandmarkCategory`). Use `contactInfo` for phone and email details, and `extraInfo` for additional metadata stored in a structured hashmap.

### Media and Images

Retrieve landmark images using `img` (primary) or `extraImg` (secondary). Validate image data before use.

### Address Information

The `address` attribute connects landmarks to `AddressInfo` for physical address details.

### Store Metadata

Attributes like `landmarkStoreId`, `landmarkStoreType`, and `timeStamp` provide information about the assigned landmark store and insertion time.

### Unique Identifier

The `id` ensures every landmark is uniquely identifiable.

If the `ContactInfo` or `ExtraInfo` object retrieved from a landmark is modified, you must use the corresponding setter to update the value associated with the landmark.

For example:
```dart
ContactInfo info = landmark.contactInfo;
info.addField(type: ContactInfoFieldType.phone, value: '5555551234', name: 'office phone');
// highlight-next-line
landmark.contactInfo = info; // <-- Does not update the value associated with the landmark without this line
```

The `ExtraInfo` object also stores data relevant for geographic area, contour geographic area, and Wikipedia information. Modifying `extraInfo` may cause data loss if related fields are not preserved.

---

## Create Landmarks

Create landmarks using one of these methods:

- **Default**: `Landmark()` - Creates a basic landmark object

- **With coordinates**: `Landmark.withLatLng(latitude, longitude)` - Creates a landmark at specific coordinates

- **With Coordinates object**: `Landmark.withCoordinates(Coordinates coordinates)` - Uses a predefined `Coordinates` object

Creating a landmark does not automatically display it on the map. See [Display landmarks](/guides/maps/display-map-items/display-landmarks) for instructions.

## Interaction with Landmarks

### Select Landmarks

Landmarks are selectable by default. User interactions like taps identify landmarks programmatically using `cursorSelectionLandmarks()`. See [Landmark selection](../maps/interact-with-map#select-landmarks) for details.

### Highlight Landmarks

Highlight landmarks to customize their visual appearance. Provide an identifier to activate, deactivate, or update highlights. Updating overrides the previous highlight. See [Highlight landmarks](/guides/maps/display-map-items/display-landmarks#highlight-landmarks) for details.

### Search Landmarks

Search landmarks by name, location, route proximity, address, and more. Filter searches by landmark categories. See [Get started with Search](/guides/search/get-started-search) for details.

### Calculate Routes

Landmarks are the primary entities for route calculations. See [Get started with Routing](/guides/routing/get-started-routing) for details.

### Proximity Alarms

Configure alarms to notify users when approaching specific landmarks. See [Landmarks and overlay alarms](/guides/alarms/landmark-and-overlay-alarms) for implementation details.

### Common Uses

- Map POIs (settlements, roads, addresses, businesses) are landmarks

- Search results return landmark lists

- Route waypoints are landmarks

---

## Landmark Categories

Landmarks are categorized based on their assigned categories. Each category is defined by a unique ID, an image (which can be used in various UI components created by the SDK user), and a name that is localized based on the language set for the SDK in the case of default categories. Additionally, a landmark may be associated with a parent landmark store if assigned to one.

A single landmark can belong to multiple categories simultaneously.

### Predefined generic categories

The default landmark categories are presented below:

| **Category**                     | **Description**                                                                |
|----------------------------------|--------------------------------------------------------------------------------|
| **gasStation**                   | Locations where fuel is available for vehicles.                                |
| **parking**                      | Designated areas for vehicle parking, including public and private lots.       |
| **foodAndDrink**                 | Places offering food and beverages, such as restaurants, cafes, or bars.       |
| **accommodation**                | Facilities providing lodging, including hotels, motels, and hostels.           |
| **medicalServices**              | Healthcare facilities like hospitals, clinics, and pharmacies.                 |
| **shopping**                     | Retail stores, shopping malls, and markets for purchasing goods.               |
| **carServices**                  | Auto repair shops, car washes, and other vehicle maintenance services.         |
| **publicTransport**              | Locations associated with buses, trains, trams, and other public transit.      |
| **wikipedia**                    | Points of interest with available Wikipedia information for added context.     |
| **education**                    | Educational institutions such as schools, universities, and training centers.  |
| **entertainment**                | Places for leisure activities, such as cinemas, theaters, or amusement parks.  |
| **publicServices**               | Government or civic buildings like post offices and administrative offices.    |
| **geographicalArea**             | Specific geographical zones or regions of interest.                            |
| **business**                     | Office buildings, corporate headquarters, and other business establishments.   |
| **sightseeing**                  | Tourist attractions, landmarks, and scenic points of interest.                 |
| **religiousPlaces**              | Places of worship, such as churches, mosques, temples, or synagogues.          |
| **roadside**                     | Features or amenities located along the side of roads, such as rest areas.     |
| **sports**                       | Facilities for sports and fitness activities, like stadiums and gyms.          |
| **uncategorized**                | Landmarks that do not fall into any specific category.                         |
| **hydrants**                     | Locations of water hydrants, typically for firefighting purposes.              |
| **emergencyServicesSupport**     | Facilities supporting emergency services, such as dispatch centers.            |
| **civilEmergencyInfrastructure** | Infrastructure related to emergency preparedness, such as shelters.            |
| **chargingStation**              | Stations for charging electric vehicles.                                       |
| **bicycleChargingStation**       | Locations where bicycles can be charged, typically for e-bikes.                |
| **bicycleParking**               | Designated parking areas for bicycles.                                         |

Find category IDs in the `GenericCategoriesId` enum. Use the `getCategory` static method from `GenericCategories` to get the `LandmarkCategory` associated with a `GenericCategoriesId` value.

In addition to the predefined categories, custom landmark categories can be created, offering flexibility to define tailored classifications for specific needs or applications.

### Category Hierarchy

Each generic category can include multiple POI subcategories. The `LandmarkCategory` class is used for both levels.

For example, the *Parking* category contains subcategories like *Park and Ride*, *Parking Garage*, *Parking Lot*, *RV Park*, *Truck Parking*, *Truck Stop*, and *Parking meter*.

Retrieve POI subcategories using `GenericCategories.getPoiCategories()`. Find the parent generic category using `GenericCategories.getGenericCategory()`.

**Important distinction:**

- `getCategory` - Returns `LandmarkCategory` object by ID

- `getGenericCategory` - Returns parent generic `LandmarkCategory` of a POI subcategory

### Category Uses

- Filter search results by category

- Toggle landmark visibility on the map

- Organize landmarks within stores

---

## Landmark Stores

**Landmark stores** are collections of landmarks and categories used throughout the SDK. Each store has a unique `name` and `id`.

Stores persist on device in a SQLite database and remain accessible across sessions.

Landmark coordinates are subject to floating-point precision limitations, which may cause positioning inaccuracies of a few centimeters to meters.

### Manage Landmark Stores

Manage landmark stores using the `LandmarkStoreService` class.

#### Create a Landmark Store

Create a new landmark store:
```dart
LandmarkStore landmarkStore = LandmarkStoreService.createLandmarkStore('MyLandmarkStore');
```

This method creates a new store or returns an existing one with the same name. Optional parameters include zoom level visibility and custom file path.

Stores persist across sessions. Creating a store with an existing name returns that store, potentially containing previous landmarks and categories.

#### Get Landmark Store by ID

Retrieve a store by its ID:
```dart
LandmarkStore? landmarkStoreById = LandmarkStoreService.getLandmarkStoreById(12345);
```

Returns the `LandmarkStore` object or `null` if the ID doesn't exist.

#### Get Landmark Store by Name

Retrieve a store by name:
```dart
LandmarkStore? landmarkStoreByName = LandmarkStoreService.getLandmarkStoreByName('MyLandmarkStore');
```

Returns the `LandmarkStore` object or `null` if the name doesn't exist.

#### Get All Landmark Stores

Retrieve all landmark stores:
```dart
List<LandmarkStore> landmarkStores = LandmarkStoreService.landmarkStores;
```

This returns both user-created and predefined SDK stores.

#### Remove Landmark Stores

Remove a landmark store:
```dart
int landmarkStoreId = landmarkStore.id;
landmarkStore.dispose();
LandmarkStoreService.removeLandmarkStore(landmarkStoreId);
```

This removes the store from persistent storage.

**Requirements:**

- Dispose the store before removing it (undisposed stores will not be removed)

- Get the store ID before disposing (operations on disposed stores throw exceptions)

- Ensure the store is not in use (displayed on map or monitored by `AlarmService`)

If the store is in use, removal fails and `ApiErrorService.apiError` is set to `GemError.inUse`.

#### Get Landmark Store Type

Retrieve the store type:
```dart
LandmarkStoreType type = LandmarkStoreService.getLandmarkStoreType(storeId);
```

#### Predefined Landmark Stores

The SDK includes predefined stores:
```dart
int mapPoisLandmarkStoreId = LandmarkStoreService.mapPoisLandmarkStoreId;
int mapAddressLandmarkStoreId = LandmarkStoreService.mapAddressLandmarkStoreId;
int mapCitiesLandmarkStoreId = LandmarkStoreService.mapCitiesLandmarkStoreId;
```

Use these IDs to determine if a landmark originated from default map elements.

**Do not modify these stores.** They are used for:

- Filtering landmark categories displayed on the map

- Checking landmark origin

- Filtering significant landmarks in search and alarms

#### Import Landmarks

Import landmarks from a file or data buffer into an existing store. Supported formats include KML and GeoJSON. Assign categories and images to imported landmarks. Monitor progress with the returned `ProgressListener`.
```dart
ProgressListener? listener = landmarkStore.importLandmarks(
  filePath: '/path/to/file',
  format: LandmarkFileFormat.kml,
  image: landmarkImage,
  onComplete: (GemError error) {
    if (error == GemError.success) {
      // Handle success
    } else {
      // Handle failure
    }
  },
  categoryId: yourCategoryId,
);
```

**Parameters:**

- `filePath` - File path of the landmark file

- `format` - File format (KML, GeoJSON). See `LandmarkFileFormat`

- `image` - Image to associate with landmarks

- `onComplete` - Callback invoked on completion with `GemError`

- `categoryId` - Category ID for imported landmarks (must be valid). Use `uncategorizedLandmarkCategId` for no category

The `categoryId` must be valid.

#### Import from Data Buffer

Import landmarks from a raw data buffer:
```dart
ProgressListener? listener = landmarkStore.importLandmarksWithDataBuffer(
  buffer: fileBytes,
  format: LandmarkFileFormat.geoJson,
  image: landmarkImage,
  onComplete: (GemError error) {
    if (error == GemError.success) {
      // Handle success
    } else {
      // Handle failure
    }
  },
  categoryId: yourCategoryId,
);
```

**Parameters:**

- `buffer` - Binary data representing the landmark file

- `format` - File format (KML, GeoJSON). See `LandmarkFileFormat`

- `image` - Map image to associate with landmarks

- `onComplete` - Callback triggered on completion with `GemError`

- `categoryId` - Category ID for imported landmarks

Use this method when receiving data as binary.

### Browse Landmark Stores

Use `LandmarkBrowseSession` to efficiently browse stores with many landmarks.

Create a browse session:
```dart
LandmarkBrowseSession browseSession = landmarkStore.createLandmarkBrowseSession(
  settings: LandmarkBrowseSessionSettings(
    // Specify the settings here
  ),
);
```

Only landmarks present in the store at session creation are available in the browse session.

**Browse Session Settings:**

| Field Name          | Type               | Default Value                           | Description |
|---------------------|--------------------|-----------------------------------------|-------------|
| `descendingOrder`   | `bool`             | `false`                                 | Specifies whether the sorting of landmarks should be in descending order. By default, sorting is ascending. |
| `orderBy`           | `LandmarkOrder`    | `LandmarkOrder.name`                    | Specifies the criteria used for sorting landmarks. By default, landmarks are sorted by name. Other options may include sorting by distance, and insertion date. |
| `nameFilter`        | `String`           | empty string                            | A filter applied to landmark names. Only landmarks that match this name substring will be included. |
| `categoryIdFilter`  | `int`              | `LandmarkStore.invalidLandmarkCategId`  | Used to filter landmarks by category ID. The default value is considered invalid, meaning all categories are matched. |
| `coordinates`       | `Coordinates`      | an invalid instance                     | Specifies a point of reference used when ordering landmarks by distance. Only relevant when `orderBy == LandmarkOrder.distance`. |

**Browse Session Operations:**

| Member | Type | Description |
|--------|------|-------------|
| `id` | `int` (getter) | Retrieves the unique ID of this session. |
| `landmarkStoreId` | `int` (getter) | Returns the ID of the associated `LandmarkStore`. |
| `landmarkCount` | `int` (getter) | Gets the total number of landmarks in this session. |
| `getLandmarks(int start, int end)` | `List<Landmark>` | Retrieves landmarks between indices `[start, end)`. Used for pagination or slicing the landmark list. |
| `getLandmarkPosition(int landmarkId)` | `int` | Returns the 0-based index of a landmark by its ID, or a not-found code. |
| `settings` | `LandmarkBrowseSessionSettings` (getter) | Gets the current session settings. Modifying this object does not affect the session. |

### Landmark Store Operations

The `LandmarkStore` class provides these operations:

<table>
  <thead>
    <tr>
      <th>Operation</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>addCategory(LandmarkCategory category)</code></td>
      <td>Adds a new category to the store. The category must have a name. After addition, the category belongs to this store.</td>
    </tr>
    <tr>
      <td><code>addLandmark</code></td>
      <td>Adds a **copy of the landmark** to a specified category in the store. Updates category info if the landmark already exists. Can specify a category. Defaults to uncategorized if no category is specified.</td>
    </tr>
    <tr>
      <td><code>getLandmark(int landmarkId)</code></td>
      <td>Retrieves the landmark with the specified landmarkId from the store. Returns null if the landmark does not exist in the store.</td>
    </tr>
    <tr>
      <td><code>updateLandmark</code></td>
      <td>Updates information about a specific landmark in the store. This does not affect the landmark's category. The landmark must belong to this store.</td>
    </tr>
    <tr>
      <td><code>containsLandmark</code></td>
      <td>Checks if the store contains a specific landmark by its ID. Returns <code>true</code> if found, <code>false</code> otherwise.</td>
    </tr>
    <tr>
      <td><code>categories</code></td>
      <td>Retrieves a list of all categories in the store.</td>
    </tr>
    <tr>
      <td><code>getCategoryById</code></td>
      <td>Fetches a category by its ID. Returns <code>null</code> if not found.</td>
    </tr>
    <tr>
      <td><code>getLandmarks</code></td>
      <td>Retrieves a list of landmarks in a specified category. Defaults to all categories if none is specified.</td>
    </tr>
    <tr>
      <td><code>removeCategory</code></td>
      <td>Removes a category by its ID. Optionally removes landmarks in the category or marks them as uncategorized.</td>
    </tr>
    <tr>
      <td><code>removeLandmark</code></td>
      <td>Removes a specific landmark from the store.</td>
    </tr>
    <tr>
      <td><code>updateCategory</code></td>
      <td>Updates a specific category's details. The category must belong to this store.</td>
    </tr>
    <tr>
      <td><code>removeAllLandmarks</code></td>
      <td>Removes all landmarks from the store.</td>
    </tr>
    <tr>
      <td><code>id</code></td>
      <td>Retrieves the ID of the landmark store.</td>
    </tr>
    <tr>
      <td><code>name</code></td>
      <td>Retrieves the name of the landmark store.</td>
    </tr>
    <tr>
      <td><code>type</code></td>
      <td>Retrieves the type of the landmark store. Can be none, defaultType, mapAddress, mapPoi, mapCity, mapHighwayExit or mapCountry. </td>
    </tr>
  </tbody>
</table>

### Landmark Store Uses

- Display landmarks on the map

- Customize search functionality

- Manage proximity alarms

- Persist landmarks across sessions

---
