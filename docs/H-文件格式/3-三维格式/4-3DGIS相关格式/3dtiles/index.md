---
title: 3dtiles
---

## 引言
> 摘自：[3dTiles 数据规范详解](https://zhuanlan.zhihu.com/p/148312375)

早年，地理的三维还处于地形三维上，即数字高程模型（DEM）提供地表的高度拉伸。

随着学科的融合、计算机技术和硬件的更新换代，使得有模型、有细节的真三维融入到GIS中成为了可能，或者说，计算机技术和硬件的升级，给GIS以更广阔的视角观察世界。

面对大规模精细三维数据的加载，还要照顾到GIS的各种坐标系统、分析计算，gltf这种单个模型的方案显得力不从心。

2016年，Cesium 团队借鉴传统2DGIS的地图规范——WMTS，借鉴图形学中的层次细节模型，打造出大规模的三维数据标准—— 3d-Tiles，中文译名：三维瓦片。

-   它在模型上利用了 gltf 渲染快的特点，对大规模的三维数据进行组织，包括层次细节模型、模型的属性数据、模型的层级数据等。

## 3dtiles应用场景
3dtiles是用于流式传输大规模异构3D地理空间数据集的开放规范。

- 为了扩展Cesium的地形和图像流，3D Tiles将用于流式传输3D内容，包括建筑物，树木，点云和矢量数据
- 例如摄影测量、3D 建筑、BIM/CAD、实例化特征和点云

## 3dtiles的组成
3DTiles由`tileset.json`和tile组成，其中tile可以是.b3dm、.i3dm、.pnts、.vctr和.cmpt中的任一种格式文件。

| 类型 | 英文名称 | 文件后缀名 | 对应实际数据 |
| --- | --- | --- | --- |
| 批量三维模型 | Batch 3D Model | [b3dm](https://www.cnblogs.com/onsummer/p/13252896.html) | 传统三维建模数据、BIM数据、倾斜摄影数据 |
| 实例三维模型 | Instance 3D Model | i3dm | 一个模型多次渲染的数据，灯塔、树木、椅子等 |
| 点云 | PointCloud | pnts | 点云数据 |
| 复合模型 | Component | cmpt | 前三种数据的复合（允许一个cmpt文件内嵌多个其他类型的瓦片） |

## 3dtiles优点
第一，3dTiles 继承了 gltf 的优点

- 贴合图形渲染 API 的逻辑，讨 GPU 喜爱，webgl 对其内部组织起来的三维模型数据，不需要转换，可以直接渲染（glTF 的功劳）

第二：3dTiles（`tileset.json`）不记录模型数据，只记录各级“Tile”的逻辑关系，以及元数据

- 所谓的模型数据，是指三维模型的顶点、贴图材质、法线、颜色等信息。而模型数据存储在.b3dm、.i3dm、.pnts、.vctr和.cmpt中的任一种格式文件
- 逻辑关系是指，各级Tile是如何在空间中保持连续的，LOD是如何组织的

第三：瓦片的意义

- 网速是有限的，在加载超大规模的三维模型数据时，不可能把一个模型全部下载下来再渲染，那样等待的时间太慢了，但是一点一点出现，视野范围外的“瓦片”则干脆就不下载、渲染，性能、视觉都有提高。这就是瓦片的设计优点
- 传统的二维地图瓦片，叫做 WMTS 或 TMS，这个 "T" 就是 Tile 的意思。

## 参考文章

1. [3dtiles格式规范-官方手册](https://github.com/CesiumGS/3d-tiles/tree/main/specification)
2. [3dtiles标准基于JSON格式指定，这是它的schema](https://github.com/CesiumGS/3d-tiles/tree/main/specification/schema)
3. [3dTiles系列博客](https://www.cnblogs.com/onsummer/p/12799366.html)
