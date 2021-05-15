# Mosaics & composites

One of the most sought after functions in Earth Engine is the possibility of using deep time stack imagery to create cloud free composites. One of the simplest way of thinking about this is to use reducers that we talked about earlier, where we look at an entire stack of pixels and choose the median value of the distribution of pixel across stack and we end up getting a cloud free composite over the given time period. Depending on the number of images, the actual number of cloud free images in the overall stack your results may need more fine tune adjustments.

The mosaics function in GEE adds the latest or more recent image on top while trying to mosaic, where composites like median or max are based on the mathematical operation by that name. Mosaics is thereby a more specific compositor which takes into account latest pixel on top first during the mosaic operation or compositing.

![ee_mosaics_composites](https://user-images.githubusercontent.com/6677629/118322451-bea4e780-b4c4-11eb-884f-012fbd1bc24c.gif)

```js
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

print('Total filter images in collection',collection.size())

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
