# Reducers & Charts in Earth Engine

While single image operations are direct and can be applied to the imagery for a one to one result. Applying any kind of analysis on a single image means you have to call the image and run the analysis which is quick and easy. To be able to turn this same into a function that you can iterate over an entire collection requires us to convert a single analysis to a function. A function is then mapped or run over an entire collection. To avoid any errors make sure that the collection images are consistent and have same name and number of band and characteristics.

<center>![Imagery](/images/ee_reduce.png)</center>
<center>**Source: [Image Collection Reductions from Google Earth Engine](https://developers.google.com/earth-engine/reducers_image_collection)**</center>

For this setup we look at how we added Landsat 5 Surface Reflectance data earlier , filtering it using WRS Path and Row and further using Cloud cover for the scene. The next step we are building an function to calculate Normalized Difference Vegetation Index (NDVI) over the entire collection which takes an image collection and iteratively passes an image to the function. The resultant structure is also an image collection where we are returning a single band which is the NDVI. Note that we also renamed the band to NDVI since they are not autorenamed. Now the additional step we added here was running the produced collection through a median reducer. We can then print the median values. The last step we create a setup for the chart and the plot the chart of the median

``` js
/*Add some field points and name the geometry
as field this script won't work till then*/

//Add an image collection
var collection=ee.ImageCollection('LANDSAT/LT05/C01/T1_SR')

//Filtering an Image Collection
var filtered=collection.filterMetadata('WRS_PATH','equals',25)
.filterMetadata('WRS_ROW','equals',39).filterMetadata('CLOUD_COVER','less_than',5)

//print filtered collection properties
print("Filtered Collection",filtered)


/*==================================================*/
//Writing a function
var addNDVI = function(image) {
  var ndvi = image.normalizedDifference(['B4', 'B3']).rename('NDVI');
  return ndvi;
};

var ndvicoll=filtered.map(addNDVI)
print("NDVI Collection",ndvicoll)

//Let's add a palette for us to show the results
var ndvivis = {"opacity":1,"bands":["NDVI"],"min":-0.5182320475578308,"max":0.7803871631622314,"palette":["d49e8a","ffcfb4","ecffcf","94ff8d","3eff56","15cc23"]};

//Add the first of the result
Map.addLayer(ee.Image(ndvicoll.first()),ndvivis,"NDVI")

//Add an reducer
var ndviav=ndvicoll.reduce(ee.Reducer.median());

Map.addLayer(ndviav.select('NDVI_median').rename('NDVI'),ndvivis,"NDVI Median")
print(ndviav)
```
<center>![ndvi](/images/ee_ndvi.JPG)</center>
<center>![chart](/images/ee_chart.JPG)</center>
<center>**Top: Field points to look at median NDVI, Bottom: Chart generated from field point**</center>

``` js
var ndvionly=ndviav.select('NDVI_median').rename('NDVI')

var chart = ui.Chart.image.byRegion({
  image: ndvionly,
  regions: field,
  scale: 30
});
chart.setOptions({
  title: 'Charting NDVI',
  vAxis: {
    title: 'NDVI'
  },
  legend: 'none',
  lineWidth: 1,
  pointSize: 4
});

print(chart);

```
