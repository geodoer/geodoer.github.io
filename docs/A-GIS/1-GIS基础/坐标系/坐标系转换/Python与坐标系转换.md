
## 示例
### 4326转4978

代码：
```python
import pyproj
lon = -74.6604194 
lat = 40.3450248
alt = 50

transformer = pyproj.Transformer.from_crs("EPSG:4326", "EPSG:4978", always_xy = True)
print(transformer)

xyz = transformer.transform(lon, lat, alt)
print('Projected XY', xyz)

lonlat = transformer.transform(xyz[0], xyz[1], xyz[2], direction='INVERSE')
print('Lon Lat', lonlat)
```

输出：
```
proj=pipeline step proj=unitconvert xy_in=deg xy_out=rad step proj=cart ellps=WGS84
Projected XY (1287785.7839034656, -4694607.645412987, 4107291.4290166595)
Lon Lat (-74.6604194, 40.3450248, 50.0)
```