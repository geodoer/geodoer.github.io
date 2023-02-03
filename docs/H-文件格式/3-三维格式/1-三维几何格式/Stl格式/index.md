---
title: STL格式
---

STL文件结构非常简单，其只能描述三维物体的几何信息，不支持颜色材质等信息，是三维打印机支持的最常见的文件格式之一。

stl文件有两种：

1.  一种是明码格式
2.  另一种是[二进制](https://so.csdn.net/so/search?q=%E4%BA%8C%E8%BF%9B%E5%88%B6&spm=1001.2101.3001.7020)格式

相关库

1.  [https://github.com/dacunni/stlloader](https://github.com/dacunni/stlloader)

## 明码STL
```
solid filename stl     //文件路径及文件名 
facet normal x y z     //三角面片法向量的3个分量值 
outer loop 
   vertex x y z        //三角面片第一个顶点坐标 
   vertex x y z        //三角面片第二个顶点坐标 
   vertex x y z        //三角面片第三个顶点坐标 
   endloop   
   endfacet            //完成一个三角面片定义 
   --- 
endsolid filename stl  //整个STL文件定义结束
```

## 二进制STL

```
UINT8[80]         -   Header
UINT32            -   Number of triangles
foreach triangle
  REAL32[3]       -    Normal vector
  REAL32[3]       -    Vertex 1
  REAL32[3]       -    Vertex 2
  REAL32[3]       -    Vertex 3
  UINT16          -    Attribute byte count
end
```
