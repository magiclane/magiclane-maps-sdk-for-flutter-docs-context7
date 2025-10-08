---
description: Documentation for Landmarks Markers Overlays
title: Landmarks Markers Overlays
---

# Landmarks vs Markers vs Overlays

When building a sophisticated mapping application, choosing the right type of object to use for your specific needs is crucial. To assist in making an informed decision, we compare the three core mapping entities in the table below:

<table>
<tr>
    <th>Characteristic</th>
    <th>Landmarks</th>
    <th>Markers</th>
    <th>Overlays</th>
</tr>
<tr>
    <td>Select from map</td>
    <td>basic selection using `cursorSelectionLandmarks()`</td>
    <td>advanced selection using `cursorSelectionMarkers()`, providing matched marker and the collection it belongs to, plus positional details such as the marker’s index within its collection, which part/segment or vertex was hit and so on</td>
    <td>basic selection using `cursorSelectionOverlayItems()`</td>
</tr>
<tr>
    <td>On the map by default</td>
    <td>yes</td>
    <td>no</td>
    <td>yes, if present within the style</td>
</tr>
<tr>
    <td>Customizable render settings</td>
    <td>basic level of customization using highlights</td>
    <td>high level of customization using MarkerRenderSettings</td>
    <td>within the style (in Studio). Also allows customization using highlights</td>
</tr>
<tr>
    <td>Visibility and layering</td>
    <td>toggleable based on the category and store</td>
    <td>can individually be changed</td>
    <td>toggleable based on the category and overlay</td>
</tr>
<tr>
    <td>Searchable</td>
    <td>yes</td>
    <td>no</td>
    <td>yes</td>
</tr>
<tr>
    <td>Can be used for route calculation</td>
    <td>yes</td>
    <td>no</td>
    <td>no</td>
</tr>
<tr>
    <td>Can be used for alarms</td>
    <td>yes</td>
    <td>no</td>
    <td>yes</td>
</tr>
<tr>
    <td>Create custom items</td>
    <td>programmatically within the client application</td>
    <td>programmatically within the client application</td>
    <td>using uploaded GeoJSON Data Sets (in Studio); they cannot be created within the client application<sup>(1)</sup></td>
</tr>
<tr>
    <td>Available offline</td>
    <td>yes</td>
    <td>yes</td>
    <td>no, with some exceptions</td>
</tr>
<tr>
    <td>Shared among users</td>
    <td>yes, only for predefined landmarks. Changes made to landmarks and custom landmarks are local</td>
    <td>no</td>
    <td>yes, the overlay items are accessible by all users given they have the correct style applied</td>
</tr>
<tr>
    <td>Extra info</td>
    <td>address contact info, category, etc.</td>
    <td>no</td>
    <td>data with flexible structure (`SearchableParameterList`)</td>
</tr>
</table>

Social reports can be created and modified by app clients and are accessible to all other users.
