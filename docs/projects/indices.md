# Collection Maps & Indices

Now that we know how to build functions like the cloud mask, what about applying a simple band ratio or an index and adding it back to the image? Mapping over a collection is powerful because it allows you to work efficiently over each image in the collection without needing to aggregate or composite before hand though both approaches can find some use.

For this example we are going to run a simple Normalized Difference Vegetation Index over our area of interest and add the results back with a visualization palette.

![ee_indices](https://user-images.githubusercontent.com/6677629/118348802-4f55e480-b512-11eb-97fd-f9eafcced252.gif)


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

print('Total filtered images in collection',collection.size())


//Add all functions in one place

// Function to remove cloud and snow pixels from Sentinel-2 SR image
function masks2(image) {
  var cloudProb = image.select('MSK_CLDPRB');
  var snowProb = image.select('MSK_SNWPRB');
  var cloud = cloudProb.lt(1);
  var snow = snowProb.lt(1);
  var scl = image.select('SCL');
  var shadow = scl.eq(3); // 3 = cloud shadow
  var cirrus = scl.eq(10); // 10 = cirrus
  // Cloud probability less than 5% or cloud shadow classification
  var mask = (cloud.and(snow)).and(cirrus.neq(1)).and(shadow.neq(1));
  return image.updateMask(mask);
}

// Write a function that computes NDVI for an image and adds it as a band
function addNDVI(image) {
  var ndvi = image.normalizedDifference(['B8', 'B4']).rename('ndvi');
  return image.addBands(ndvi);
}

var palette = [
  'FFFFFF', 'CE7E45', 'DF923D', 'F1B555', 'FCD163', '99B718',
  '74A901', '66A000', '529400', '3E8601', '207401', '056201',
  '004C00', '023B01', '012E01', '011D01', '011301'];

var ndviVis = {min:0, max:0.5, palette: palette }

collection = collection.map(masks2).map(addNDVI)
print('Single Image post NDVI',collection.first())

/*
Things to keep in mind, image collections are usually sorted by default based on date
Mosaic function adds the latest pixels or most recent image on top while trying to mosaic
Median composite on the other hands takes the median value of pixel over the time period
*/

var medianComposite = collection.median();
Map.addLayer(ee.ImageCollection(collection).median().select("ndvi").clip(geometry),ndviVis,'NDVI Median')
```
