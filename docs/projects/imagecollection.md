# Image Collections in Earth Engine

While single images are great to do quick analytics, the true power of the Earth Engine environment comes with the possibility of looking at really large and heavy image collections and to be able to push analysis towards the data, rather than the need for the data to travel at all. In the GEE environment image collections have their own characteristic setup and are composted with single images that we discussed earlier. They can often have the same or different band structure but generally share a similar metadata structure for filtering and querying.

Large scale image collections such as Landsat and Sentinel image collections are ingested on the fly and are actively maintained till there imagery and processing pipelines feeds are maintained byt he agencies supplying the imagery. Images as well as image collections can be moved into GEE environment to allow you to use both your data and the GEE catalog data within the same platform.

For those who are concerned with access to datasets, this means that though Earth Engine allows an easier way to share datasets across users, private folder, collections and imagery are private and are not here the section from their [Terms and Conditions page](https://earthengine.google.com/terms/)

**```Intellectual Property Rights. Except as expressly set forth in this Agreement,
this Agreement does not grant either party any rights, implied or otherwise,
to the other’s content or any of the other’s intellectual property.
As between the parties, Customer owns all Intellectual Property Rights
in Customer Data, Customer Code, and Application(s), and Google owns
all Intellectual Property Rights in the Services and Software.```**

<center>![ee_image_collection](https://user-images.githubusercontent.com/6677629/118183847-3957ff00-b400-11eb-8ba7-03b83b421229.gif)</center>

These image collection as well as individual imaegs again have defined data type,scales and projections along with some default properties such as an index and ID among other system properties. So we can query these properties, print them and add them

``` js
var s2 = ee.ImageCollection("COPERNICUS/S2_SR");
var geometry =
    ee.Geometry.Polygon(
        [[[-89.79297430041262, 29.677212347812258],
          [-89.79297430041262, 28.850584616352855],
          [-88.91681463244387, 28.850584616352855],
          [-88.91681463244387, 29.677212347812258]]], null, false);
var vis = {"opacity":1,"bands":["B8","B3","B2"],"min":1,"max":2870,"gamma":1.4140000000000001};

//Let's constrain the Sentinel-2 SR collection by our geometry & a cloudy pixel percentage metadata
var collection = s2
.filter(ee.Filter.bounds(geometry))
.filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE',5))

print(collection.size())  //Prints the size of the collection
//Remember the collection has been constrained to our geometry & cloudy pixel %

//Print & add the first element from the collection
Map.centerObject(geometry,10)
print('First image from Collection',collection.first())
Map.addLayer(collection.first().clip(geometry),vis,'First image from Collection')
```

To have a look at all of the raster catalog you can find them [listed here](https://code.earthengine.google.com/datasets)
