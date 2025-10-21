---
description: Documentation for Landmarks
title: Landmarks
---

# Landmarks

A **landmark** is a predefined, permanent location that holds detailed information such as its name, address, description, geographic area, categories (e.g., Gas Station, Shopping), entrance locations, contact details, and sometimes associated multimedia (e.g., icons or images). It represents significant, categorized locations with rich metadata, providing structured context about a place.

## Landmark Structure  

#### Geographic Details 

A landmark's position is defined by its `coordinates`, which represent the centroid, and its `geographicArea`, representing the full boundary (e.g., circle, rectangle, or polygon). Since landmarks can correspond to buildings, roads, settlements, or regions, the geographic area can be complex. For specific bounding areas, the `getContourGeographicArea` method is used.

#### Descriptive Information  

Landmarks include attributes like `name`, `description`, and `author`. The name adapts to the SDK's language settings, ensuring localization where applicable.

#### Metadata  

Landmarks can belong to one or more `categories`, described by `LandmarkCategory`. Additional details like `contactInfo` (e.g., phone, email) and `extraInfo` (a structured hashmap) add flexibility for storing metadata.

#### Media and Imagery 

Images associated with landmarks can be retrieved using getters like `img` (primary image) or `extraImg` (secondary images). Please note that these images might contain invalid data and it is the user's responsability to check the validity of the objects using the provided methods.

#### Advanced Metadata  

Attributes such as `landmarkStoreId`, `landmarkStoreType` provide information about assigned landmark store, the landmark store type. The ``timeStamp`` records information about the time the landmark was inserted into a store.

**Address**  
The `address` attribute connects landmarks to `AddressInfo`, providing details about the physical address of the location.

#### Unique Identifier

The `id` ensures every landmark is uniquely identifiable.

If the `ContactInfo` or `ExtraInfo` object retrieved from a landmark is modified, you must use the corresponding setter to update the value associated with the landmark.

For example:
```dart
ContactInfo info = landmark.contactInfo;
info.addField(type: ContactInfoFieldType.phone, value: '5555551234', name: 'office phone');
// highlight-next-line
landmark.contactInfo = info; // <-- Does not update the value associated with the landmark without this line
```

The `ExtraInfo` object also stores data relevant for the geographic area, contour geographic area and the wikipedia related information.

Modifying the `extraInfo` field of the landmark may lead to the loss of this information if the related fields are not preserved.

## Instantiating Landmarks  

Landmarks can be instantiated in multiple ways:  
1. **Default Initialization**: `Landmark()` creates a basic landmark object.  
2. **Using Latitude & Longitude**: `Landmark.withLatLng(latitude, longitude)` ties a landmark to geographic coordinates.  
3. **With a Coordinates Object**: `Landmark.withCoordinates(Coordinates coordinates)` uses a predefined `Coordinates` object.  

Creating a new landmark does not automatically make it visible on the map. Refer to the [Display landmarks](/guides/maps/display-map-items/display-landmarks) guide for detailed instructions on how to display a landmark.

## Interaction with Landmarks

### Selecting landmarks

Landmarks are selectable by default, meaning user interactions, such as taps or clicks, can identify specific landmarks programmatically (e.g., through the function `cursorSelectionLandmarks()`). Refer to the [Landmark selection guide](/guides/maps/interact-with-map#landmark-selection) for more details. 

### Highlighting landmarks

A list of landmarks can be highlighted, enabling customization of their visual appearance.
Highlighting also allows displaying a list of user created landmarks on the map. When displaying them, we can also provide an identifier for the highlight. It is possible to de-activate that highlight or to update it - in this case the old highlight is overridden. Refer to the [Highlight landmarks](/guides/maps/display-map-items/display-landmarks#highlight-landmarks) guide for more details.

### Searching landmarks

Landmarks are searchable (both landmarks from map and user created landmarks). Search can be performed based on name, geographic location, proximity to a given route, address, and more. Options to filter the search based on landmark categories are available. Refer to the [Get started with Search](/guides/search/get-started-search) guide for more details.

### Calculating route with landmarks

Landmarks are the sole entities used for route calculations.
For detailed guidance, refer to the [Get started with Routing](/guides/routing/get-started-routing) guide.

### Get notifications when approaching landmarks

Alarms can be configured to notify users when approaching specific landmarks that have been custom-selected.
See the [Landmarks and overlay alarms](/guides/alarms/landmark-and-overlay-alarms) guide for more details about implementing this feature.

## Other usages

- Most map POIs (such as settlements, roads, addresses, businesses, etc.) are landmarks. 

- Search results return a list of landmarks.

- Intermediary points in a Route are landmarks.

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

The id for each category can be found in the ``GenericCategoriesId`` enum, each value having assigned an ``id``. Use the `getCategory` static method from the `GenericCategories` class to get the `LandmarkCategory` class associated with a `GenericCategoriesId` value.

In addition to the predefined categories, custom landmark categories can be created, offering flexibility to define tailored classifications for specific needs or applications.

### Tree structure

Each **generic landmark category** can include multiple **poi subcategories**. The `LandmarkCategory` is used both for generic categories and poi subcategories.

For example, the *Parking* generic category contains *Park and Ride*, *Parking Garage*, *Parking Lot*, *RV Park*, *Truck Parking*, *Truck Stop* and *Parking meter* poi subcategories.

To retrieve poi subcategories for a generic category, use the `getPoiCategories` static method provided in the `GenericCategories` class.
To find the parent generic category of a given subcategory use the `getGenericCategory` static method provided in the `GenericCategories` class.

Do not confuse the `getCategory` and `getGenericCategory` methods:

- The `getCategory` is used to get the `LandmarkCategory` object based on the id

- The `getGenericCategory` is used to get the *parent generic* `LandmarkCategory` object of the poi subcategory with the provided id.

### Usage

- Can be used as a filter parameter within the various types of search.

- Landmark visibility on the map can be toggled based on the categories. 

- Landmark organization within a store

## Landmark Stores

**Landmark stores** are the most common collection of landmarks used for multiple purposes through the Maps SDK for Flutter. They are comprised of landmarks and landmarks categories. Each store has a unique `name` and `id`.

Landmark stores are persistently saved on the device (a SQLite database file) and can be accessed across different sessions.

### Manage landmark stores

The operations related to LandmarkStore management can be found within the `LandmarkStoreService` class.

#### Create a new landmark store

A new landmark store can be created in the following way:
```dart
LandmarkStore landmarkStore = LandmarkStoreService.createLandmarkStore('MyLandmarkStore');
```

The ``createLandmarkStore`` will create a new landmark store with the given name if it does not already exist or will return the existing landmark store if an instance with the given name already exists. The method also provides optional parameters to set the zoom level at which the landmarks contained within are visible on the map and a custom file path for storing the landmark on the file system can also be provided.

Since the landmark store is persistent, the `createLandmarkStore` method may return a `LandmarkStore` instance that already contains landmarks and categories from previous sessions, if a store with the same name was created before.

#### Get landmark store by id

A landmark store can be obtained by its id:
```dart
LandmarkStore? landmarkStoreById = LandmarkStoreService.getLandmarkStoreById(12345);
```

The `getLandmarkStoreById` method retrieves a valid `LandmarkStore` object when provided with a valid ID. If the ID does not correspond to any `LandmarkStore`, the method returns null.

#### Get landmark store by name

A landmark store can be obtained by its name:
```dart
LandmarkStore? landmarkStoreByName = LandmarkStoreService.getLandmarkStoreByName('MyLandmarkStore');
```

The `getLandmarkStoreByName` method returns a valid `LandmarkStore` object if a landmark store exists with the given name and null if the given name does not correspond to any `LandmarkStore`.

#### Get all landmark stores

The list of landmark stores can be retrieved via the ``landmarkStores`` getter:
```dart
List<LandmarkStore> landmarkStores = LandmarkStoreService.landmarkStores;
```

The ``landmarkStores`` getter returns both user created landmark stores and predefined stores in the Maps SDK for Flutter.

#### Remove landmark stores

A landmark store can be removed in the following way:
```dart
int landmarkStoreId = landmarkStore.id;
landmarkStore.dispose();
LandmarkStoreService.removeLandmarkStore(landmarkStoreId);
```

This will also remove the store from the persistent storage.

Disposing the ``LandmarkStore`` object is mandatory before calling the ``removeLandmarkStore`` method. If the store is not disposed then it **will not be removed**.
Any operation called on a disposed ``LandmarkStore`` instance will result in an exception being thrown. Therefore, it is crucial to obtain the landmark store ID before it is disposed.

The ``removeLandmarkStore`` method will fail if the store is in use (for example, if it is displayed on the map or monitored within the ``AlarmService``). In this case, the ``ApiErrorService.apiError`` will be set to `GemError.inUse`. 

#### Retrieving the landmark store type

The type of a ``LandmarkStore`` can be returned by calling ``LandmarkStoreService.getLandmarkStoreType`` with the store ID.

#### Predefined landmark stores

The Maps SDK for Flutter includes several predefined landmark stores, and their IDs can be retrieved as follows:
```dart
int mapPoisLandmarkStoreId = LandmarkStoreService.mapPoisLandmarkStoreId;
int mapAddressLandmarkStoreId = LandmarkStoreService.mapAddressLandmarkStoreId;
int mapCitiesLandmarkStoreId = LandmarkStoreService.mapCitiesLandmarkStoreId;
```

These values can be useful when determining whether a landmark originated from the default map elements. If its associated `landmarkStoreId` has one of these 3 values, it means it comes from the map rather than from other custom data.

These three landmark stores are not intended to be modified. They are only used for functionalities like:

- Filter the categories of landmarks displayed on the map.

- Check whether a landmark comes from the map.

- When searching, to filter the significant landmarks.

- When using alarms, to filter the significant landmarks.

#### Import landmark stores

You can import a landmark store using data from a file or a raw data buffer. 

The `importLandmarks` and `importLandmarksWithDataBuffer` methods import data into an existing `LandmarkStore`. Before calling these methods, you must first create or retrieve the target LandmarkStore. The import can be performed from a supported file format (e.g., KML, GeoJSON) or directly from a raw data buffer. You can assign a category and a custom image to the imported landmarks. A ProgressListener is returned to monitor the import progress, and an optional callback can be used to handle completion and error states.
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

- `filePath`: The file path of the landmark file to import.

- `format`: The file format (e.g., KML, GeoJSON), see `LandmarkFileFormat`.

- `image`: An image to associate with the landmarks.

- `onComplete`: A callback invoked once the import finishes, providing a `GemError`.

- `categoryId`: The category ID for the imported landmark store. Must refer to an existing category. Use `uncategorizedLandmarkCategId` to import without a category.

- The `categoryId` must be valid.

#### Import from data buffer

Landmark stores can also be imported from a raw data buffer:
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

- `buffer`: The binary data representing the landmark file content.

- `format`: The file format (e.g., KML, GeoJSON), see `LandmarkFileFormat`.

- `image`: A map image to associate with the landmarks.

- `onComplete`: A callback triggered upon completion with a `GemError`.

- `categoryId`: The category ID for the imported landmarks.

:::tip

Use this method when you receive the data as a binary.

### Browse landmark stores

The `LandmarkBrowseSession` class provides an efficient way to browse and search within a `LandmarkStore` that contains a large number of landmarks.

To create a `LandmarkBrowseSession`, use the `createLandmarkBrowseSession` method available in the `LandmarkStore` class. This method takes a `LandmarkBrowseSessionSettings` object as an argument, which defines the sorting and filtering criteria for the session.
```dart
LandmarkBrowseSession browseSession = landmarkStore.createLandmarkBrowseSession(
  settings: LandmarkBrowseSessionSettings(
    // Specify the settings here
  ),
);
```

Only the landmark contained within the store before the session creation are available in the browse session.

The available sorting and filter settings are:

| Field Name          | Type               | Default Value                           | Description |
|---------------------|--------------------|-----------------------------------------|-------------|
| `descendingOrder`   | `bool`             | `false`                                 | Specifies whether the sorting of landmarks should be in descending order. By default, sorting is ascending. |
| `orderBy`           | `LandmarkOrder`    | `LandmarkOrder.name`                    | Specifies the criteria used for sorting landmarks. By default, landmarks are sorted by name. Other options may include sorting by distance, and insertion date. |
| `nameFilter`        | `String`           | empty string                            | A filter applied to landmark names. Only landmarks that match this name substring will be included. |
| `categoryIdFilter`  | `int`              | `LandmarkStore.invalidLandmarkCategId`  | Used to filter landmarks by category ID. The default value is considered invalid, meaning all categories are matched. |
| `coordinates`       | `Coordinates`      | an invalid instance                     | Specifies a point of reference used when ordering landmarks by distance. Only relevant when `orderBy == LandmarkOrder.distance`. |

The operations available within the `LandmarkBrowseSesssion` are:

| Member | Type | Description |
|--------|------|-------------|
| `id` | `int` (getter) | Retrieves the unique ID of this session. |
| `landmarkStoreId` | `int` (getter) | Returns the ID of the associated `LandmarkStore`. |
| `landmarkCount` | `int` (getter) | Gets the total number of landmarks in this session. |
| `getLandmarks(int start, int end)` | `List<Landmark>` | Retrieves landmarks between indices `[start, end)`. Used for pagination or slicing the landmark list. |
| `getLandmarkPosition(int landmarkId)` | `int` | Returns the 0-based index of a landmark by its ID, or a not-found code. |
| `settings` | `LandmarkBrowseSessionSettings` (getter) | Gets the current session settings. Modifying this object does not affect the session. |

### Available operations

The ``LandmarkStore`` instance provides the following operations for managing landmarks and categories:

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

### Usage

- Landmarks are displayed on the map through landmark stores.

- Landmark stores are used to customize search functionality.

- Alarms for approaching landmarks are managed through landmark stores.

- Landmark stores are used for persisting landmarks between different sessions.
