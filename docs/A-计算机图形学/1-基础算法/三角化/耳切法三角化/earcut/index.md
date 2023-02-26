---
title: earcut(三角化库)
---

## 相关地址

1. [https://github.com/mapbox/earcut.hpp](https://github.com/mapbox/earcut.hpp)
2. [Earcut: 最快和最小的JavaScript多边形三角剖分库](https://www.5axxw.com/wiki/content/24lbwr)

## 简介
该库实现了一个改进的耳朵切片算法，通过z-order曲线哈希优化，并扩展到处理孔、扭曲多边形、退化和self-intersections，这种方法不保证三角剖分的正确性，但始终尝试为实际数据生成可接受的结果。

它基于FIST的思想：Martin Hold的快速Industrial-Strength多边形三角剖分和David Eberly的耳朵裁剪三角剖分。

该项目的目标是创建一个JS三角剖分库，该库足够快速地在浏览器中进行real-time三角剖分，牺牲三角剖分质量以获得原始速度和简单性，同时具有足够的健壮性来处理大多数实际数据集，而不会崩溃或产生垃圾。

它最初创建的用例是mapboxgl，WebGL-based交互式地图。  
如果你想得到正确的三角剖分，即使在非常糟糕的数据上有大量的self-intersections，那么看看libtess.js（earcut是不够精确）。

## 示例代码

```cpp
// The number type to use for tessellation
using Coord = double;

// The index type. Defaults to uint32_t, but you can also pass uint16_t if you know that your
// data won't have more than 65536 vertices.
using N = uint32_t;

// Create array
using Point = std::array<Coord, 2>;
std::vector<std::vector<Point>> polygon;

// Fill polygon structure with actual data. Any winding order works.
// The first polyline defines the main polygon.
polygon.push_back({{100, 0}, {100, 100}, {0, 100}, {0, 0}});
// Following polylines define holes.
polygon.push_back({{75, 25}, {75, 75}, {25, 75}, {25, 25}});

// Run tessellation
// Returns array of indices that refer to the vertices of the input polygon.
// e.g: the index 6 would refer to {25, 75} in this example.
// Three subsequent indices form a triangle. Output triangles are clockwise.
std::vector<N> indices = mapbox::earcut<N>(polygon);
```