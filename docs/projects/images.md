# Images in Earth Engine

In the GEE environment images are stored in Cloud Optimized Geospatial tiles instead of a single image which allows for running an analysis this scale. This means that though the input imagery comes in know formats such as geotiff , MrSid and img these datasets post ingestion into GEE are converted into tiles that are used for at scale analysis. All images that are ingested into either GEE(s) Raster Catalog or your own personal folder and stored in folder or collections of images as you would expect to see when doing deep time stack analysis.

<center>![ee_image](https://user-images.githubusercontent.com/6677629/118183847-3957ff00-b400-11eb-8ba7-03b83b421229.gif)</center>

These images have defined data type,scales and projections along with some default properties such as an index and ID among other system properties. So we can query these properties, print them and add them

``` js
var s2 = ee.ImageCollection("COPERNICUS/S2_SR");
var geometry =
    ee.Geometry.Polygon(
        [[[-89.79297430041262, 29.677212347812258],
          [-89.79297430041262, 28.850584616352855],
          [-88.91681463244387, 28.850584616352855],
          [-88.91681463244387, 29.677212347812258]]], null, false);
var vis = {"opacity":1,"bands":["B8","B3","B2"],"min":1,"max":2870,"gamma":1.4140000000000001};
var image = ee.Image('COPERNICUS/S2_SR/20190831T162839_20190831T164212_T16RBT')
print('Single Image',image)

//Center the Map to the image and add the image
Map.centerObject(geometry,10)
Map.addLayer(image,vis,"S2-SR Image 2019-08-31")

//Clip an image
var clipped=image.clip(geometry)
Map.addLayer(clipped,vis,"Clipped S2-SR Image 2019-08-31")
```
