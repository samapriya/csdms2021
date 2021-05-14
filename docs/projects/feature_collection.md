# Feature Collections in Earth Engine

Feature Collections are similar to Image Collections - but they contain Features, not images. They are equivalent to Vector Layers in a GIS. We can load, filter and display Feature Collections using similar techniques that we have learned so far. Similar to bringing in your own images and creating image collections, it is possible to upload shapefiles and CSV tables directly into earh engine enriching the list of datasets made available to you.

For this example we chose to use the Hydrologic Unit Code or HUC boundaries already preingested in Google Earth Engine. I decided to use HUC 6 but you can choose a scale depending on your question of interest.

``` js
var s2 = ee.ImageCollection("COPERNICUS/S2_SR");
var geometry =
    ee.Geometry.Polygon(
        [[[-89.79297430041262, 29.677212347812258],
          [-89.79297430041262, 28.850584616352855],
          [-88.91681463244387, 28.850584616352855],
          [-88.91681463244387, 29.677212347812258]]], null, false);
var vis = {"opacity":1,"bands":["B8","B3","B2"],"min":1,"max":2870,"gamma":1.4140000000000001};
var HUC6 = ee.FeatureCollection("USGS/WBD/2017/HUC06");
var image = ee.Image('COPERNICUS/S2_SR/20190831T162839_20190831T164212_T16RBT')

print('Total HUC 6 Features',HUC6.size())

//Add all HUC6 features that intersects with our geometry & create a transparent overlay
Map.addLayer(HUC6.filterBounds(geometry).style({fillColor: '00000000',color: 'FF5500'}),{},'HUC 6 Transparent Overlay',false);

//Now clip the image we first added using the feature collection
Map.addLayer(image.clip(HUC6.filterBounds(geometry)),vis,"HUC 6 Clipped S2-SR Image 2019-08-31",false)
```
