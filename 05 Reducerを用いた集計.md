# Reducerを用いた集計


## Reducerの概念
ReducerはGEEの強力な機能の一つです．
Reducerを使うと膨大な量のデータについて，空間・時間・バンドなどを単位とした集計が可能になります．

ここでは，例として平成27年7月九州北部豪雨によって生じた土砂災害ポリゴンごとに，豪雨前後のNDVIがどのくらい変化をしているのか集計します．

## Reducerの手順

Reducerの手順は以下になります．

1. 集計される側のImageCollectionを用意する
2. 集計単位のFeatureCollectionを用意する
2. Reducerを用いて集計する

## 豪雨前後のNDVIの最大値を取得する

豪雨前後のNDVIの最大値を求めます．
平成27年7月九州北部豪雨は2017年7月5日から6日にかけて起きました．
この日付を境として前後1年ずつのImageCollectionを作成します．
`start1`と`end1`は豪雨前の開始日と終了日，
`start2`と`end2`を豪雨後の開始日と終了日とします．
豪雨前と豪雨後の期間はそれぞれ約1年としました．

```javascript
// 平成29年7月九州北部豪雨（2017年7月5日〜6日）前
var start1 = ee.Date('2016-07-01');
var end1 = ee.Date('2017-07-04');

// 平成29年7月九州北部豪雨後
var start2 = ee.Date('2017-07-07');
var end2 = ee.Date('2018-07-04');

var lon = 132;
var lat = 33;
var point = ee.Geometry.Point(lon, lat);
```

```javascript
// 平成29年7月九州北部豪雨（2017年7月5日〜6日）前
var start1 = ee.Date('2015-07-01');
var end1 = ee.Date('2017-06-30');

// 平成29年7月九州北部豪雨後
var start2 = ee.Date('2017-07-07');
var end2 = ee.Date('2019-06-30');

var lon = 132;
var lat = 33;
var point = ee.Geometry.Point(lon, lat);

// 豪雨前の衛星画像の取得
var ImageCollection1 = ee.ImageCollection('LANDSAT/LC08/C01/T1_SR')
    .filterDate(start1, end1)
    .filter(ee.Filter.eq('WRS_PATH', 112))
    .filter(ee.Filter.eq('WRS_ROW', 37));

// 豪雨後の衛星画像の取得
var ImageCollection2 = ee.ImageCollection('LANDSAT/LC08/C01/T1_SR')
    .filterDate(start2, end2)
    .filter(ee.Filter.eq('WRS_PATH', 112))
    .filter(ee.Filter.eq('WRS_ROW', 37));


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
var ndvidiff = ndvimax1.subtract(ndvimax2);

var landslides = ee.FeatureCollection("users/morusaevo9/20170810asakura_toho_handokuzu")
  .filter(ee.Filter.inList('name', ['土砂崩壊地', '洪水流到達範囲']));

print(landslides);

// 土砂災害ポリゴンごとのNDVI差分の集計
var landslides = ndvidiff.reduceRegions({
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

// マップ中心座標の設定
Map.setCenter(lon, lat, 10);

// NDVIと差分の表示パラメーターの定義
var ndviparam = {
  min: -1,
  max: 1,
  palette: ['blue', 'white', 'green']
};

var diffparam = {
  min: -1,
  max: 1,
  palette: ['blue', 'white', 'red']
};

Map.addLayer(ndvimax1, ndviparam, 'max NDVI before landslides');
Map.addLayer(ndvimax2, ndviparam, 'max NDVI after landslides');
Map.addLayer(ndvidiff, diffparam, 'max NDVI diff');
Map.addLayer(landslides.filter(ee.Filter.eq('name', '土砂崩壊地')), {color: 'orange'}, '土砂崩壊地');
Map.addLayer(landslides.filter(ee.Filter.eq('name', '洪水流到達範囲')), {color: 'blue'}, '洪水流到達範囲');


```
