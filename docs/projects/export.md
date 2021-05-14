# Export Images in Earth Engine

When we are all said and done we still want to export the images. Google Earth Engine allows you to export images externally into two subsystems, a Google Cloud Storage Bucket(Free quota upto 5 GB) or Google Drive (This is tied to your overall quota). The method we are exploring right now is export to Google Drive, and then being able to import the analyzed image into any local tool or libraries. It is possible to export entire collections to drive using batch exports in the python API client. This avoids the need for you to click on the Run button everytime an export task has to be started.

<center>![export](/images/ee_export.JPG)</center>
<center>**Export Window: Export Image to Google Drive**</center>

For this setup we look at how we added Landsat 5 Surface Reflectance data earlier , filtering it using WRS Path and Row and further using Cloud cover for the scene. The next step we are building an function to calculate yearly composites from 1984 to 2010 using the Landsat 5 data. Note that this is another way of creating a function where we have inserted the map function and the collection inside the function so it can be run directly. We might be interested in sorting these collections using year and hence we set the year as a metadata for each image in the cloud free composite. Depending on the type of imagery another good way of creating cloud free composites is by using the pixel qa bit bands in the Landsat surface reflectance imagery. The last step that is added is the Export to drive function, where we set up the image name, the image type, the scale and the region refers to areas over which we are exporting this imagery.

``` js

//Add an image collection
var collection=ee.ImageCollection('LANDSAT/LT05/C01/T1_SR')

//Filtering an Image Collection
var filtered=collection.filterMetadata('WRS_PATH','equals',25)
.filterMetadata('WRS_ROW','equals',39).filterMetadata('CLOUD_COVER','less_than',15)

//print filtered collection properties
print("Filtered Collection",filtered)

//Create Multi Year Composite from Landsat 5 Surface Reflectance
var years = ee.List.sequence(1984, 2010)

var multiyear = ee.ImageCollection(years
  .map(function(y) {
    var start = ee.Date.fromYMD(y, 1, 1)
    var end = start.advance(1, 'year');
    var image = filtered.filterDate(start, end).median();
    return image.set('year', y)
}))

print(multiyear);

//Add a visualization
var vis = {"opacity":1,"bands":["B4","B3","B2"],"min":-95.56918120427508,"max":2171.008347369839,"gamma":1};

//Add the Image
Map.addLayer(ee.Image(ee.ImageCollection(multiyear).first()),vis,"Median from MultiYear")

//Export Imagery
Export.image.toDrive({
  image:ee.Image(ee.ImageCollection(multiyear).first())
  .select(['B1','B2','B3','B4','B5','B6','B7']).toUint16(),
  description: "Median-Image-Export",
  folder: 'EE-CSDMS-Test',
  scale:30,
  region: geometry,
  maxPixels:10e12
})
```

Exporting video seems like the obvious thing to do after this. For this a couple of things to keep in mind, you can only export 3 band videos cast to a 8 bit image per frame. You can sort these images before export and that is the ordering of the frames.

``` js
// Load and format the collection to export.
var frame = ee.ImageCollection(multiyear)
  .sort('year')
  .select(['B4', 'B3', 'B2'])
  .map(function(image) {
    return image.visualize({bands: ['B4', 'B3', 'B2'], min: -100, max:2200})
    .set('year',image.get('year'));
  });

// Export video
Export.video.toDrive({
  collection: frame,
  folder: 'EE-CSDMS-Test',
  description: 'EE-CSDMS-Video',
  dimensions: 1080,
  framesPerSecond: 1,
  region: geometry
});
```
