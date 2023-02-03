相关链接

1. [查看epsg编码](https://epsg.io/)


## What
EPSP

- 英文全称是：European Petroleum Survey Group
- 中文名称是：欧洲石油调查组织
- 这个组织成立于1986年，2005年并入IOGP(International Association of Oil & Gas Producers)，中文名称为国际油气生产者协会
- EPSG对世界的每一个地方都制定了地图，但是由于坐标标系不同，所以地图也各不相同。

## 常用的EPSG

| EPSG | 坐标系 | 说明 |
| --- | --- | --- |
| 4326 | WGS84的经纬度坐标系 | 由美国主导的GPS系统就是在用它。WGS(World Geodetic System)是世界大地测量系统的意思，由于是1984年定义的，所以叫WGS84 |
| 3857 | WGS 84 / Pseudo-Mercator | 各大互联网地图公司以它为基准，例如Google地图，Microsoft地图都在用它 |
| 32650 | WGS 84 / UTM zone 50N | 厦门区域的投影坐标系（笔者读书时经常用） |
| 4978 | WGS84的空间直接坐标系 | 用经纬度记录数据的WGS84坐标系，WKID是4326。用地心为坐标原点的**空间直角坐标**来记录数据的坐标系，WKID是4979。 |

## 参考链接

1. [EPSG是什么？](https://www.zhihu.com/question/52220968)