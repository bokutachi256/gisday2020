### NDVIの計算

NDVIは以下の式で求まります．

$$
NDVI = \frac{NIR-Red}{NIR+Red}
$$

ここで，$Red$は赤の波長の反射率，$NIR$は近赤外の波長の反射率です．
Lansast-8では，バンド4（B4）が赤の反射率，バンド5（B5）が近赤外の反射率です．
`image`に対して`select`を使ってバンド4とバンド5を取り出します．

```javascript
var red = image.select('B4');
var nir = image.select('B5');
```

```javascript
var ndvi = nir.subtract(red)
  .divide(nir.add(red))
  .rename('NDVI');
```

```javascript
var ndvi = image.normalizedDifference(['B5', 'B4']).rename('NDVI');

print(image);

var ndviParams = {
  min: -1,
  max: 1,
  palette: ['blue', 'white', 'green']
  };

Map.addLayer(ndvi, ndviParams, 'NDVI');
```

#### NDVIの計算プログラムの全体
```javascript
var start = ee.Date('2017-07-08');
var end = ee.Date('2017-12-31');
var lon = 132;
var lat = 33;
var point = ee.Geometry.Point(lon, lat);

var ImageCollection = ee.ImageCollection('LANDSAT/LC08/C01/T1_SR')
  .filterDate(start, end)
  .filterBounds(point)
  .sort("CLOUD_COVER", true); //propertiesのCLOUD_COVER値でソートする（true:昇順、false:降順）

var image = ImageCollection.first();

//var lsfeature = ee.Feature('users/morusaevo9/asakura');
var visparam = {
  bands: ['B4', 'B3', 'B2'],
  min: 0,
  max: 3000,
  gamma: 1.4
};

Map.setCenter(lon, lat, 10);
Map.addLayer(image, visparam, 'Landsat');

var nir = image.select('B5');
var red = image.select('B4');

var ndvi = nir.subtract(red).divide(nir.add(red)).rename('NDVI');
var ndvi = image.normalizedDifference(['B5', 'B4']).rename('NDVI');

print(image);

var ndviParams = {
  min: -1,
  max: 1,
  palette: ['blue', 'white', 'green']
  };

Map.addLayer(ndvi, ndviParams, 'NDVI');
```
