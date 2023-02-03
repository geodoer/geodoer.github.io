---
title: IfcEngine
---

> 笔者：IfcEngine是面向过程的，有很多坑，慎入。

IfcEngine

1.  ifcengine-latest：IFCEngine的库和头文件
2.  ifcengine(build 1259)：IFCEngine官方示例代码

IFCEngine引擎有两个部分

1.  解析几何的模块`#include "engine.h"`
2.  解析属性的模块名`#include "ifcengine.h"`

## IfcEngine提供的小工具

ifcengine提供了很多小工具，小工具是开源的。有两个是比较常用的

1.  ifceditor用于查看IFC实体包含的属性名，取IFC的实体属性需要通过属性名去取，因此借助这个工具可以找到要取的属性名，源码可以学习如何解析IFC实体的属性；
2.  ifcviewer用于查看IFC的几何模型，通过DirectX渲染，是BIMVision的简易版，源码可以学习如何解析IFC的几何模型