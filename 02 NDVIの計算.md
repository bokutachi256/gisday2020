### NDVIの計算

NDVIは以下の式で求まります．

NDVI is given by the following formula.

![NDVI = \frac{NIR - Red}{NIR + Red}
](https://render.githubusercontent.com/render/math?math=%5CLarge+%5Cdisplaystyle+NDVI+%3D+%5Cfrac%7BNIR+-+Red%7D%7BNIR+%2B+Red%7D%0A)

ここで，Redは赤の波長の反射率，NIRは近赤外の波長の反射率です．
Lansast-8では，バンド4（B4）が赤の反射率，バンド5（B5）が近赤外の反射率です．
`image`に対して`select`を使ってバンド4とバンド5を取り出します．

Here, Red is the reflectance at red wavelengths and NIR is the reflectance at near-infrared wavelengths.
In Lansast-8, band 4 (B4) is the red reflectance and band 5 (B5) is the near-infrared reflectance.
Use `select` on `image` to extract band 4 and 5.

```javascript
var red = image.select('B4');
var nir = image.select('B5');
```

次に赤バンドと近赤外バンドの比演算を行います．
`nir`から`subtract`メソッドを使って`red`の値を引きます．
更に`nir`と`red`の和`add`で割り`divide`，
結果を`ndvi`に代入します．

The next step is to calculate the ratio between the red and near-infrared bands.
Subtract the `red` value from `nir` using the `subtract` method.
Furthermore, divide the result by the sum of `nir` and `red` `add` and assign the result to `ndvi`.

```javascript
var ndvi = nir.subtract(red)
  .divide(nir.add(red))
  .rename('NDVI');
```

もしくはオブジェクト`image`に`noralizedDfference`を適用して比演算を直接行うこともできます．
以下のステートメントは上記の比演算と同じ結果になります．

Or you can apply the `noralizedDfference` to the object `image` and do the ratio operation directly.
The following statement gives the same result as the above ratio operation.

```javascript
var ndvi = image.normalizedDifference(['B5', 'B4']).rename('NDVI');
```

確認のためオブジェクト`ndvi`をコンソールに出力し，プロパティ`ndvi`が計算されているか確認します．

``` javascript
print(ndvi);
```

最後に求まった`ndvi`を表示します．
ndviの表示プロパティとして`ndviParams`を設定します．
NDVIは理論的には-1から1の間の値を取ります．`max`と`min`にそれぞれ-1と1を設定します．


```javascript
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

var ndvi = image.normalizedDifference(['B5', 'B4']).rename('NDVI');

print(image);

var ndviParams = {
  min: -1,
  max: 1,
  palette: ['blue', 'white', 'green']
  };

Map.addLayer(ndvi, ndviParams, 'NDVI');
```
