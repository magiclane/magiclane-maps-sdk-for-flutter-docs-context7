---
description: Documentation for Markers
title: Markers
---

# Markers

A **marker** is a visual representation (such as an icon or a geometry, like a polyline or polygon) placed at a specific geographic location on a map to indicate an important point of interest, event, or location.

Markers can represent temporary or user-specified points on the map, such as user-defined locations, waypoints, or temporary annotations. While they are often represented by icons, they can also take the form of more complex geometries, like lines or shapes, depending on the context or requirements.

Markers typically contain only basic metadata, such as their position, title, or description, without extensive associated details.

By default, the map does not include any visual elements categorized as markers. Users have the ability to create and add markers to the map as needed.

## Instantiating Markers

Markers can be instantiated via:  
1. **Default Initialization**: `Marker()` creates a basic marker object.  

Creating a marker does not automatically display it on the map. Ensure you set its coordinates and attach it to the desired map. Refer to the [Display markers guide](/guides/maps/display-map-items/display-markers) for detailed instructions.

## Marker Structure

A marker can contain multiple coordinates, which can be organized into different parts. If no part is specified, the coordinates are added to a default part, indexed as 0. Each part is rendered differently based on the marker type.

### Types of Markers

There are 3 types of markers:

- **Point markers** (each part is a group of points - array of coordinates)

- **Polyline markers** (each part is a polyline - array of coordinates)

- **Polygon markers** (each part is a polygon - array of coordinates)

The marker has methods for managing and manipulating markers on a map, including operations such as adding, updating, and deleting coordinates or parts.

A marker can be rendered in multiple ways on the map, either through default settings or user-specified rendering options:

- An image icon

- A polygon drawn with a specific color, with a specific fill color, etc.

- A polygon having an associated image at each point

## Customization options

Markers offer extensive customization options on the map, enabling developers to tailor their appearance and behavior. Customizable features include:

- **Colors**: Modify the fill color, contour color, and text color to match the desired style.

- **Sizes**: Adjust dimensions such as line width, label size, and margins to fit specific requirements.

- **Labeling and Positioning**: Define custom labeling modes, reposition item or group labels, and adjust the alignment of labels and images relative to geographic coordinates.

- **Grouping Behavior**: Configure how multiple markers are grouped when located in proximity.

- **Icons**: Customize icons for individual markers or groups, including options for image fit and alignment.

- **Polyline and Polygon Textures**: Apply unique textures to polylines and polygons for enhanced visualization.
 
**MarkerSketches** are some predefined collections in the view. For each marker type, there is such a collection. Each element of the collection has a different render settings object. 

Obviously, the regular list of markers has a more efficient rendering.
-->

## Interaction with Markers

### Selecting markers

Markers are selectable by default, meaning user interactions, such as taps or clicks, can identify specific markers programmatically (e.g., through the function `cursorSelectionMarkers()`).

The result is a list of matches. The match contains detailed information about the match:

- the marker type

- the collection of the marker

- the marker index in the collection

- the part index inside the marker

### Searching markers

Markers are **not searchable**.

### Calculating route with marker

Markers are **not** designed for route calculation.

To enable route calculation and navigation, create a new landmark using the relevant coordinates of the marker and a representative name and use that object for routing.

## MarkerCollection

The `MarkerCollection` class is the main collection holding markers. All the markers within a collection have the same type and are styled in the same way. 

### MarkerCollection structure and operations

<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Type</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>`id`</td>
      <td>int</td>
      <td>Retrieves the collection's unique ID.</td>
    </tr>
    <tr>
      <td>`clear()`</td>
      <td>void</td>
      <td>Deletes all markers from the collection.</td>
    </tr>
    <tr>
      <td>`add(Marker marker, {int index = -1})`</td>
      <td>void</td>
      <td>Adds a marker to the collection at a specific index (default is the end of the collection).</td>
    </tr>
    <tr>
      <td>`indexOf(Marker marker)`</td>
      <td>int</td>
      <td>Returns the index of a given marker in the collection.</td>
    </tr>
    <tr>
      <td>`delete(int index)`</td>
      <td>void</td>
      <td>Deletes a marker from the collection by index.</td>
    </tr>
    <tr>
      <td>`area`</td>
      <td>RectangleGeographicArea</td>
      <td>The geographic area enclosing all markers in the collection.</td>
    </tr>
    <tr>
      <td>`getMarkerAt(int index)`</td>
      <td>Marker</td>
      <td>Returns the marker at a specific index or an empty marker if the index is invalid.</td>
    </tr>
    <tr>
      <td>`getMarkerById(int id)`</td>
      <td>Marker</td>
      <td>Retrieves a marker by its unique ID.</td>
    </tr>
    <tr>
      <td>`getPointsGroupHead(int id)`</td>
      <td>Marker</td>
      <td>Retrieves the head of a points group for a given marker ID.</td>
    </tr>
    <tr>
      <td>`getPointsGroupComponents(int id)`</td>
      <td>List&lt;Marker&gt;</td>
      <td>Retrieves the components of a points group by its ID.</td>
    </tr>
    <tr>
      <td>`name`</td>
      <td>String</td>
      <td>The name of the marker collection.</td>
    </tr>
    <tr>
      <td>`size`</td>
      <td>int</td>
      <td>Returns the number of markers in the collection.</td>
    </tr>
    <tr>
      <td>`type`</td>
      <td>MarkerType</td>
      <td>Retrieves the type of the marker collection.</td>
    </tr>
  </tbody>
</table>

### Instantiating MarkerCollections

A marker collection is created by providing the name and the marker type:
```dart
MarkerCollection markerCollection = MarkerCollection(markerType: MarkerType.point, name: "myCollection");
```

### Usage

The `MarkerCollection` class is used to display markers on the map.
