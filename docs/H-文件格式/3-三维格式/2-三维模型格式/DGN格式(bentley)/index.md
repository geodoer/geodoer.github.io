---
title: bentley数据格式
---

> 内容来自ChatGPT，[√]已验证是正确信息，[×]错误信息，其他的未验证

## DGN
DGN（Design）格式是Bentley软件自主三维数据格式

- 该格式是一种基于二进制的文件格式，用于存储CAD和GIS数据，并支持复杂的二维和三维几何元素、属性和关系数据
- DGN格式最初是由Intergraph公司开发，后来被Bentley Systems采用并成为其主要的三维数据交换格式之一

## 软件
支持DGN的软件如下。需要注意的是

1. 不同的软件、工具、开发包和第三方库对DGN格式的支持程度可能会有所不同，某些特定版本可能支持的功能也会存在差异
2. 在选择使用时，请仔细查看它们的文档和功能列表，并根据自己的需求进行选择

| 软件 | 简介 | 
| - | - |
| Bentley MicroStation | Bentley Systems开发的CAD/BIM软件，能够直接读取、编辑和创建DGN格式文件 | |
|Autodesk AutoCAD | Autodesk公司开发的CAD软件，可以通过插件或者其他方式实现对DGN格式文件的读取|
| Siemens PLM Solid Edge | Siemens PLM Software开发的机械设计软件，可以读取和编辑DGN格式文件 |
| Esri ArcGIS | Esri开发的地理信息系统软件，可以读取和处理DGN格式的地图数据 |
| FME | Safe Software公司开发的数据转换和集成工具，能够读取、转换和导出多种CAD和GIS格式，包括DGN格式|


## 命令行工具

| 命令行工具 | 简介 | 收费|
| - | -|  - |
| bentley MicroStation msbatch  | 安装bentley软件后，在安装目录下有msbatch，但文档很少。 |
| ~~ODA Teigha File Converter 4.7以上版本~~<br/>([×]没有找到4.7版本，其他版本不支持读入DGN) | Open Design Alliance提供的免费命令行工具，可以实现多种CAD和BIM格式之间的相互转换，包括DGN格式。用户可以使用该工具将DGN文件转换为其他CAD/BIM格式，例如DWG、DXF、IGES和STEP等。| 免费软件，无需注册或激活 |
| FME Desktop | 可以通过命令行模式实现对DGN格式文件的读取、转换和导出|

## 开发包

| 开发包 | 简介 | 收费 |
| - | - | - |
| Open Design Alliance Teigha | Open Design Alliance提供的CAD开发包，可以读取和写入多种CAD格式，其中包括DGN格式| 收费 |
| Bentley DGN SDK |Bentley Systems提供的专门针对DGN格式的软件开发工具包，包括C++ API和.NET API。该SDK可用于读取、编辑和创建DGN文件，并提供了丰富的文档和示例代码| 收费|
| GDAL/OGR | 开源的地理空间数据处理库，可以读取、转换和处理多种GIS格式，包括DGN格式|

