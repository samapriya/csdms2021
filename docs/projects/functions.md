# Functions & Mapping

While single image operations are direct and can be applied to the imagery for a one to one result. Applying any kind of analysis on a single image means you have to call the image and run the analysis which is quick and easy. To be able to turn this same into a function that you can iterate over an entire collection requires us to convert a single analysis to a function. A function is then mapped or run over an entire collection. To avoid any errors make sure that the collection images are consistent and have same name and number of band and characteristics.

![ee_functions](https://user-images.githubusercontent.com/6677629/118319001-c1e9a480-b4bf-11eb-9275-9ee94770d4a1.gif)


For this setup we look at how we could apply cloud masking to Sentinel-2 using the Snow and Cloud Probability bands.

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

print('Total filter images in collection',collection.size())


//Introducing functions & Mapping over functions

// Function to remove cloud and snow pixels from Sentinel-2 SR image
function masks2(image) {
  var cloudProb = image.select('MSK_CLDPRB');
  var snowProb = image.select('MSK_SNWPRB');
  var cloud = cloudProb.lt(1);
  var snow = snowProb.lt(1);
  var scl = image.select('SCL');
  var shadow = scl.eq(3); // 3 = cloud shadow
  var cirrus = scl.eq(10); // 10 = cirrus
  // Cloud probability less than 1% or cloud shadow classification
  var mask = (cloud.and(snow)).and(cirrus.neq(1)).and(shadow.neq(1));
  return image.updateMask(mask);
}

collection = collection.map(masks2)

/*
Things to keep in mind, image collections are usually sorted by default based on date
Mosaic function adds the latest pixels or most recent image on top while trying to mosaic
Median composite on the other hands takes the median value of pixel over the time period
*/

var mosaic = collection.mosaic()
var medianComposite = collection.median();

Map.addLayer(collection, rgbVis, 'Filtered Collection');
Map.addLayer(mosaic.clip(geometry), rgbVis, 'Mosaic',false);
Map.addLayer(medianComposite.clip(geometry), rgbVis, 'Median Composite',false)
```
