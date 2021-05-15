# Exporting Image & Visualization

When we are all said and done we still want to export the images. Google Earth Engine allows you to export images externally into two subsystems, a Google Cloud Storage Bucket or Google Drive. The method we are exploring right now is export to Google Drive, and then being able to import the analyzed image into any local tool or libraries. It is possible to export entire collections to drive using batch exports in the python API client. This avoids the need for you to click on the Run button every time an export task has to be started.

For this setup we are going to export the Sentinel-2 mosaic imagery at a 20m resolution

![ee_exporting](https://user-images.githubusercontent.com/6677629/118347255-54f9fd00-b507-11eb-9236-18d7ff0fcd73.gif)


``` js
var s2 = ee.ImageCollection("COPERNICUS/S2_SR");
var geometry =
    ee.Geometry.Polygon(
        [[[-89.79297430041262, 29.677212347812258],
          [-89.79297430041262, 28.850584616352855],
          [-88.91681463244387, 28.850584616352855],
          [-88.91681463244387, 29.677212347812258]]], null, false);
var rgbVis = {"opacity":1,"bands":["B4","B3","B2"],"min":1,"max":1506,"gamma":1.786};
//********************************* Image Collection*****************************************//
Map.centerObject(geometry,10)

//Let's constrain the Sentinel-2 SR collection by our geometry & a cloudy pixel percentage metadata
var collection = s2
.filter(ee.Filter.bounds(geometry))
.filter(ee.Filter.date('2020-01-01', '2021-01-01'))
.filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE',5))
.select('B.*') //Also let us only add optical bands wildcard to filter only bands starting with B

print('Total filtered images in collection',collection.size())

/*
Things to keep in mind, image collections are usually sorted by default based on date
Mosaic function adds the latest pixels or most recent image on top while trying to mosaic
*/

var mosaic = collection.mosaic()

Map.addLayer(collection, rgbVis, 'Filtered Collection');
Map.addLayer(mosaic.clip(geometry), rgbVis, 'Mosaic',false);

//Using the visualize function means you have forced the image to be visualized in a specific way
var visualized = mosaic.clip(geometry).visualize(rgbVis)

//You can export the visualized image so that you get something that looks similar to how it would appear in GEE
Export.image.toDrive({
    image: visualized.clip(geometry),
    description: 'Export-Median-Composite-Visualized',
    folder: 'csdms2021',
    fileNamePrefix: 'median_composite_visualized',
    region: geometry,
    scale: 20,
    maxPixels: 1e12
})

//Or you can export the image only
Export.image.toDrive({
    image: mosaic.clip(geometry),
    description: 'Export-Median-Composite',
    folder: 'csdms2021',
    fileNamePrefix: 'median_composite',
    region: geometry,
    scale: 20,
    maxPixels: 1e12
})
```
