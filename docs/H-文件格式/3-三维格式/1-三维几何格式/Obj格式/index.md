---
title: Obj格式
---

相关链接

1.  [Object Files (.obj)](http://paulbourke.net/dataformats/obj/)
2.  [opengl教程：模型加载](http://www.opengl-tutorial.org/beginners-tutorials/tutorial-7-model-loading/)
3.  [obj文件格式详细介绍](https://www.jianshu.com/p/f7f3e7b6ebf5)
4.  [wiki/obj](https://en.wikipedia.org/wiki/Wavefront_.obj_file)

## 简介
The Wavefront Object File (.obj) Format。这是一个文本文件，它把空间中的一堆顶点、法线、纹理坐标分开来表示，然后再把它们组织起来

obj模型内部以文本存储

-   `.obj`文件存储几何造型
-   `.obj`内可能会链接一个材质库文件`.mtl`，`.mtl`内部可能会链接一些材质信息（例如`.png`图片等）

## 示例
```
# Blender3D v249 OBJ File: untitled.blend
# www.blender3d.org

mtllib cube.mtl

v 1.000000 -1.000000 -1.000000
v 1.000000 -1.000000 1.000000
v -1.000000 -1.000000 1.000000
...
vt 0.748573 0.750412
vt 0.749279 0.501284
vt 0.999110 0.501077
...
vn 0.000000 0.000000 -1.000000
vn -1.000000 -0.000000 -0.000000
vn -0.000000 -0.000000 1.000000
...

usemtl Material_ray.png
s off
f 5/1/1 1/2/1 4/3/1
f 5/1/1 4/3/1 8/4/1
f 3/5/2 7/6/2 8/7/2
f 8/11/7 7/12/7 6/10/7
...
```

注释

1.  `#是注释标记
2.  `usemtl`和`mtlib`描述模型的外观，表示材质相关的数据
3.  `o` 这里没有，它表示引入一个新的object
4.  `v`代表顶点
5.  `vt`代表顶点的纹理坐标
6.  `vn`代表顶点的法向量
7.  `f`代表面

解释：`f 8/11/7 7/12/7 6/10/7`

1.  `8/11/7`描述三角形的第一个顶点
	1. `8`表示顶点索引（索引从1开始，并不是像C++从0开始），在这里即是`-1.000000 1.000000 -1.000000`
	2. `11`表示纹理索引（索引从1开始），即`0.748355 0.998230`
	3. `7`表示法线索引（索引从1开始），即`0.000000 1.000000 -0.000000`
2. `7/12/7`描述三角形的第二个顶点
3.  `6/10/7`描述三角形的第三个顶点

我们称这个为数字索引。若几个顶点共用一个坐标，索引就显得很方便，文件中只需保存一个`v`，可以多次引用，节省存储空间。缺点：无法让OpenGL混用顶点、纹理和法线索引

## IO库

| 第三方库 | 语言 |  说明 |
| - | - | - |
| [tinyobjloader](https://github.com/tinyobjloader/tinyobjloader) |C++ | 开源、轻量化。支持读obj，内部也提供写obj的示例，但不完整 |
| assimp | C++ | |