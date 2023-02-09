---
title: PBR
---

## 引言

> PBR更像是一种概念，而不是一套严格的规则。——Joe Wilson

## 简介
PBR就是Physically-Based Rendering的缩写，意为基于物理的渲染。

- 它是计算机图形学中的一种方法，旨在模拟现实世界中光和物体的真实交互，以达到一种逼真的、真实的渲染效果
- Blinn-Phong实际上是根据现实世界的现象总结出来的经验模型，这与现实世界的物理原理很不搭。而PBR与物理原理非常契合，它使得艺术家创建材质更加容易，渲染结果也更像现实世界


## 两种工作流

【PBR材质有两种工作流程】

| 名称 | 参数值（或贴图） | 共有贴图 |
| - | - | -|
| 1.基于金属的工作流<br/>2.Metallic/Roughness<br/>3.金属/粗糙度 | 基色(BaseColor)<br/>金属度(Metallic)<br/>粗糙度(Roughness) | 法线(Normal)<br/>环境光遮蔽(Ambient Occlusion)<br/>高度(Height) |
| 1.基于镜面反射的工作流<br/>2.非金属工作流<br/>3.Specular/Glossiness<br/>4.镜面反射/光泽度| 漫反射(diffuse)<br/>镜面反射(specular)<br/>光泽度(glossiness) | 同上|

【PS】不要被名字所迷惑。基于镜面反射工作流也可以做金属，只是它做出来的金属，有可能没有比金属工作流来的真实

【两种工作流的对比图】

![](images/PBR两种工作流对比.png)

### Base Color和Diffuse(Albedo)

【Base Color】基础色，包含了两种信息

1. 金属的反射率值
2. 非金属的颜色信息（在这个案例中，非金属即是铁锈）

【Albedo(Diffuse)】反照率

1. 当金属度为0时，Albedo就代表漫反射
2. 当金属度为1的，Albedo就代表金属的反射率值  
因此非金属工作流，还需要加一个蒙版，这个蒙版就是金属度的值。

### Roughness和Glossiness

两者实际为一体两面，互为相反

【Roughness】粗糙度，白色代表粗糙 ，黑色代表光滑  
【Glossiness】光泽度，白色代表着光滑，黑色代表着粗糙

### Metallic和Specular

【Metallic】金属度，白色代表金属，黑色代表非金属  
【Specular】镜面反射率

## 参考链接

1. [The PBR Guide - Part 2 (adobe.com)](https://substance3d.adobe.com/tutorials/courses/the-pbr-guide-part2-zh)
2. [PBR材质用法及讲解](https://www.bilibili.com/video/BV1jg411n7vA)

