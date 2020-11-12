```javascript
var start = ee.Date('2017-01-01');
var end = ee.Date('2017-12-31');
var lon = 132;
var lat = 33;
var point = ee.Geometry.Point(lon, lat);

var ImageCollection = ee.ImageCollection('LANDSAT/LC08/C01/T1_SR')
  .filterDate(start, end)
  .filterBounds(point)
  .sort("SENSING_TIME", true);

print(ImageCollection);


// NDVI計算用の関数定義
var addNDVI = function(image) {
  var ndvi = image.normalizedDifference(['B5', 'B4']).rename('NDVI');
  return image.addBands(ndvi);
};

// 豪雨前後の画像画像コレクションにNDVIを追加
var withNDVI = ImageCollection.map(addNDVI);

print(withNDVI);

// 豪雨前後のNDVIの最大値の差分を計算
var ndvimax = withNDVI.select('NDVI').max();

var ndviparam = {
  min: -1,
  max: 1,
  palette: ['blue', 'white', 'green']
};

Map.setCenter(lon, lat, 10);
Map.addLayer(ndvimax, ndviparam, 'Max NDVI');

print(ui.Chart.image.series(withNDVI.select('NDVI'),
  geometry.buffer(500),
  ee.Reducer.mean(), 30));

```
