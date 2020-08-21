# denchikzika
I am currently working on this project "Spatio-temporal dynamics of arboviral diseases in Brazil in a changing climate".


This Javascript creates a NDVI cloud-free image on Google Earth Engine. 

![NDVIcloudfree](https://github.com/enoch26/denchikzika/edit/master/Screenshot (6).png)

```
// brazil municaipality shapefile                  
var brazil = ee.FeatureCollection('users/enochsuen26/brazil2010r');

// use the inspector tool to find the São Paulo state -> 645 municipality
// no new municipal btw 2010 and 2015
var muni = brazil.filterMetadata('estado_id', 'equals', '26');

//load images for composite
var collection= ee.ImageCollection('LANDSAT/LC08/C01/T1_SR')
.filterBounds(muni)
.filterDate('2019-01-01','2019-12-31').sort('CLOUD_COVER',false);

// Temporally composite the images with a maximum value function.
var visParams = {bands: ['B4', 'B3', 'B2'], min:0, max: 1000};
Map.centerObject(muni, 5);
Map.addLayer(collection, visParams, 'max value composite');

var getQABits = function(image, start, end, newName) {
    // Compute the bits we need to extract.
    var pattern = 0;
    for (var i = start; i <= end; i++) {
       pattern += Math.pow(2, i);
    }
    // Return a single band image of the extracted QA bits, giving the band
    // a new name.
    return image.select([0], [newName])
                  .bitwiseAnd(pattern)
                  .rightShift(start);
};

// A function to mask out cloudy pixels.
var cloud_shadows = function(image) {
  // Select the QA band.
  var QA = image.select(['pixel_qa']);
  // Get the internal_cloud_algorithm_flag bit.
  return getQABits(QA, 3,3, 'cloud_shadows').eq(0);
  // Return an image masking out cloudy areas.
};

// A function to mask out cloudy pixels.
var clouds = function(image) {
  // Select the QA band.
  var QA = image.select(['pixel_qa']);
  // Get the internal_cloud_algorithm_flag bit.
  return getQABits(QA, 5,5, 'Cloud').eq(0);
  // Return an image masking out cloudy areas.
};

var maskClouds = function(image) {
  var cs = cloud_shadows(image);
  var c = clouds(image);
  image = image.updateMask(cs);
  return image.updateMask(c);
};

var NDVI = function(image) {
  return image.normalizedDifference(['B5', 'B4']).rename('NDVI');
};

var composite_free = collection.map(maskClouds);
var ndvi = collection.map(maskClouds).map(NDVI);
print(ndvi.limit(20))

// set the map view and zoom level
Map.centerObject(muni,5);
Map.addLayer(composite_free, visParams, 'composite collection without clouds');
Map.addLayer(ndvi,{min: -1, max: 1, palette: ['blue', 'white', 'green']}, 'NDVI cloud free'); 

```
