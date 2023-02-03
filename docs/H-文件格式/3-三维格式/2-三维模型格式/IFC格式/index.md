---
title: IFC格式
---

IFC，Industry Foundation Classes工业基础类。

## IFC标准
IFC是一个标准，指定IFC标准的组织不提供解析库，市面上有很多IFC的解析库，其中IfcEngine比较权威。

IFC标准用`EXPRESS`语言描述

-   `EXPRESS`语言的后缀名.exp
-   一个IFC标准对应一个EXP（放在Tools\IFCTo3dTiles\bin\x64\Debug\目录下，IfcEngine使用对应版本的exp解析文件）

IFC版本

1.  `IFC2X3_TC1.exp`
2.  `IFC4.exp`；（IFC4_ADD1.exp；IFC4_ADD2.exp是4的小版本）
3.  `IFC4X1.exp`

IFC把实体、实体的几何结构、关系、属性都抽象成了对象。可以说，IFC本身是一个对象数组。

## Ifc数据结构
IFC对象间（描述一个空间结构）的树状关系：最多为四层  

1. IfcProject项目  
2. IfcSite场地  
3. IfcBuilding楼房  
4. IfcBuildingStorey（对象的最小单元）  
  
构件（IfcBuildingStorey的一个个部件），包含了几何、属性、颜色、材质  

1. 构件替代物：自定义的构件  
  
实体间的关系是用关系类来表示的，关系有很多，而且对于不同实体间用不同的关系类表示，关系类也支持自定义扩展。  

1. 空间结构  
2. 引用结构  
3. 包含结构  

## 相关资料

1.  [IFC基础](https://www.yuque.com/docs/share/25c33983-3adf-4c4b-bb1d-ce3c6ecce7fd?#%20%E3%80%8AIFC%E5%9F%BA%E7%A1%80%E3%80%8B)
2.  [IFC几何模型](https://blog.csdn.net/liyazhen2011/article/details/80562497)
3.  [IFC(Industry Foundation Classes)学习笔记一](https://zhuanlan.zhihu.com/p/94434790)
4.  [IFCEngine笔记](https://www.yuque.com/docs/share/be712361-0dd9-4410-9a93-d17a754d8dfb?#%20%E3%80%8AIFCEngine%E3%80%8B)