## GIS Day

### Google Earth Engine


```javascript
var start = ee.Date('2017-07-08');
var finish = ee.Date('2017-12-31');
var lon = 132;
var lat = 33;
var point = ee.Geometry.Point(lon, lat);

var landslides = ee.FeatureCollection("users/morusaevo9/20170810asakura_toho_hanokuzu")
  .filter(ee.Filter.eq('name', '土砂崩壊地'));

var ImageCollection = ee.ImageCollection('LANDSAT/LC08/C01/T1_SR')
    .filterDate(start, finish)
    .filterBounds(point)
    .sort("CLOUD_COVER", true); //propertiesのCLOUD_COVER値でソートする（true:昇順、false:降順）

var image = ImageCollection.first();

//var lsfeature = ee.Feature('users/morusaevo9/asakura');

Map.setCenter(lon, lat, 10);
print(image);
//print(image.properties.DATE_ACQUIRED)
Map.addLayer(image, {bands: ['B4', 'B3', 'B2'], min: 0, max: 3000}, 'Landsat');
Map.addLayer(landslides, {color: 'FF0000'}, 'Landslide');
print(landslides);

// Add reducer output to the Features in the collection.
var landslidestats = image.reduceRegions({
  collection: landslides,
  reducer: ee.Reducer.mean(),
  scale: 30,
});
print(landslidestats);

Export.table.toDrive({
  collection: landslidestats,
  description:'landslide',
  fileFormat: 'GeoJSON'
});
```

## test
