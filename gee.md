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

## 今のところ最新版
```javascript
// 平成29年7月7月九州北部豪雨（2017年7月5日〜6日）


// 平成29年7月九州北部豪雨前
var start1 = ee.Date('2015-07-01');
var finish1 = ee.Date('2017-06-30');

// 平成29年7月九州北部豪雨後
var start2 = ee.Date('2017-07-20');
var finish2 = ee.Date('2019-06-30');

// 中心の座標
var lon = 132;
var lat = 33;
var point = ee.Geometry.Point(lon, lat);

// 土砂崩壊ポリゴンデータの読み込み
var landslides = ee.FeatureCollection("users/morusaevo9/20170810asakura_toho_hanokuzu");

// 土地被覆データの読み込み
var dataset = ee.Image('ESA/GLOBCOVER_L4_200901_200912_V2_3');
var landcover = dataset.select('landcover');

// 豪雨前の衛星画像の取得
var ImageCollection1 = ee.ImageCollection('LANDSAT/LC08/C01/T1_SR')
    .filterDate(start1, finish1)
    .filter(ee.Filter.eq('WRS_PATH', 112))
    .filter(ee.Filter.eq('WRS_ROW', 37));

// 豪雨後の衛星画像の取得
var ImageCollection2 = ee.ImageCollection('LANDSAT/LC08/C01/T1_SR')
    .filterDate(start2, finish2)
    .filter(ee.Filter.eq('WRS_PATH', 112))
    .filter(ee.Filter.eq('WRS_ROW', 37));

// NDVIの計算方法もろもろ
//var nir = image.select('B5');
//var red = image.select('B4');
//var ndvi = nir.subtract(red).divide(nir.add(red)).rename('NDVI');
//var ndvi = image.normalizedDifference(['B5', 'B4']).rename('NDVI');

// NDVI計算用の関数定義
var addNDVI = function(image) {
  var ndvi = image.normalizedDifference(['B5', 'B4']).rename('NDVI');
  return image.addBands(ndvi);
};

// 豪雨前後の画像画像コレクションにNDVIを追加
var withNDVI1 = ImageCollection1.map(addNDVI);
var withNDVI2 = ImageCollection2.map(addNDVI);

// 豪雨前後のNDVIの最大値の差分を計算
var ndvimax1 = withNDVI1.select('NDVI').max();
var ndvimax2 = withNDVI2.select('NDVI').max();
var diff = ndvimax1.subtract(ndvimax2);

// SRTM DEMの読み込みと傾斜の計算
var dataset = ee.Image("JAXA/ALOS/AW3D30/V2_2");
var elevation = dataset.select('AVE_DSM');
var slope = ee.Terrain.slope(elevation);

// マップ中心座標の設定
Map.setCenter(lon, lat, 10);

// NDVIと差分の表示パラメーターの定義
//var ndviParams = {min: -1, max: 1, palette: ['blue', 'white', 'green']};
var ndviParams = {min: 0, max: 1, palette: ['white', 'green']};
var diffParams = {min: -0.5, max: 0.5, palette: ['blue', 'white', 'red']};
var palette = ['FF0000', '00FF00', '0000FF', 'FFFF00', 'FF00FF'];
var lsParams = {name: palette, max: 14};

// 計算結果の表示
//Map.addLayer(slope, {min: 0, max: 60}, 'slope');
//Map.addLayer(ndvimax1, ndviParams, 'max NDVI before landslide');
//Map.addLayer(ndvimax2, ndviParams, 'max NDVI after landslide');
Map.addLayer(diff, diffParams, 'diff');
//Map.addLayer(landslides.filter(ee.Filter.eq('name', '土砂崩壊地')), {color: 'orange'}, '土砂崩壊地');
//Map.addLayer(landslides.filter(ee.Filter.eq('name', '洪水流到達範囲')), {color: 'blue'}, '洪水流到達範囲');
//Map.addLayer(landslides, {color: 'FF8800'}, 'landslides');
//Map.addLayer(landcover, {}, 'Landcover');

// 計算結果の表示
print(landslides);
print(withNDVI1);
print(withNDVI2);

// 土砂災害ポリゴンごとのNDVI差分の集計
// Add reducer output to the Features in the collection.
var landslides = diff.reduceRegions({
  collection: landslides,
  reducer: ee.Reducer.mean(),
  scale: 30,
});
// 計算した平均値をndvimeanとして加える
var landslides = landslides.map(function(feature){
  return feature.set({ndvimean: feature.get('mean')});
});

// 集計結果集計結果の表示
print('landslides', landslides);

// 土砂災害ポリゴンごとの斜面傾斜の集計
var landslides = slope.reduceRegions({
  collection: landslides,
  reducer: ee.Reducer.mean(),
  scale: 30,
});

// 計算した平均値をslopemeanとして加える
var landslides = landslides.map(function(feature){
  return feature.set({slopemean: feature.get('mean')});
});

// 洪水流到達範囲のcategoryを0，土砂崩壊地のcategoryを1にする
var landslides = landslides.map(function(feature){
  return feature.set({category: feature.get('name')});
});
var landslides =
  landslides.remap(['土砂崩壊地', '洪水流到達範囲', '判読不能範囲'], [0, 1, 2], 'category');

 // category が1以下のみを残す
var landslides = landslides.filter(ee.Filter.lte('category', 1));

// NDVIとSlopeの散布図を出力する
var chart = ui.Chart.feature.groups(
    landslides, 'slopemean', 'ndvimean', 'name'
  )
  .setChartType('ScatterChart')
  .setOptions({
    hAxis: {title: 'Average Slope Angle (deg)'},
    vAxis: {title: 'Average difference of Maximum NDVI'},
  });
//  .setSeriesNames(["ski", "don't ski"]);
print(chart);

print(landcover)

// 集計結果をCSVで保存する
Export.table.toDrive({
  collection: landslides,
  description:'landslides',
  fileFormat: 'CSV'
});
```
