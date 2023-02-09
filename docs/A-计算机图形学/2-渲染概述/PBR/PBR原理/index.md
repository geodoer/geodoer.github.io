---
title: PBR原理
---

## “基于物理的着色”，而非“基于物理着色”

PBR，Physically Based Rendering，基于物理的渲染，它仍然只是对基于物理原理的现实世界进行一种 **近似**，这也就是为什么它被称为 ”基于物理的着色“（Physically based shading），而非”物理着色“（Physical shading）的原因。

判断一种PBR光照模型是否是基于物理的，必须满足以下三个条件

1.  基于微平面(Microfacet)的表面模型
2.  能量守恒
3.  应用基于物理的BRDF

PBR遵循的是一种被称为 反射率方程(The Reflectance Equation)的特化版本。要正确地理解PBR，很重要的一点就是要首先透彻地理解反射率方程。

### 能量守恒

PBR为了实现物理学上的可信度，它遵守了能量守恒定律，即 反射光线的总和 永远不能超过 入射光线的总量。

严格上来说，同样采用$\omega_i$（入射光方向）和$\omega_o$（出射方向）作为输入参数的 Blinn-Phong光照模型也被认为是一个BRDF。然而由于Blinn-Phong模型并没有遵循能量守恒定律，因此它不被认为是基于物理的渲染。


### 微平面模型
PBR采用了“微平面模型”的理论，其反射率方程也是基于“微平面模型”所构建的。  
具体请参阅：《微平面模型》

### 反射率方程
PBR的计算公式是“反射率方程”。  
具体请参阅：《反射率方程》

## 延伸阅读

-   [Background: Physics and Math of Shading by Naty Hoffmann](http://blog.selfshadow.com/publications/s2013-shading-course/hoffman/s2013_pbs_physics_math_notes.pdf)：由于在这一篇文章中要谈论的理论点太多，所以这里的理论知识都只是涉及到了皮毛。如果你希望了解更多关于光线背后的物理知识以及它们和PBR理论之间有什么关联的话，**这**才是你需要阅读的资源。
-   [Real shading in Unreal Engine 4](http://blog.selfshadow.com/publications/s2013-shading-course/karis/s2013_pbs_epic_notes_v2.pdf)：探讨了Epic Games在他们的Unreal 4引擎中所采用的PBR模型。我们这些教程中主要涉及的PBR系统就是基于他们的PBR模型。
-   [Marmoset: PBR Theory](https://www.marmoset.co/toolbag/learn/pbr-theory)：主要针对美术师的PBR介绍，不过仍然是很好的读物。
-   [Coding Labs: Physically based rendering](http://www.codinglabs.net/article_physically_based_rendering.aspx)：介绍渲染方程以及它和PBR直接的关系。
-   [Coding Labs: Physically Based Rendering - Cook–Torrance](http://www.codinglabs.net/article_physically_based_rendering_cook_torrance.aspx)：介绍了Cook-Torrance BRDF.
-   [Wolfire Games - Physically based rendering](http://blog.wolfire.com/2015/10/Physically-based-rendering)：介绍了PBR，由Lukas Orsvärn所著。

## 参考链接

1. [PBR理论](https://learnopengl-cn.github.io/07%20PBR/01%20Theory/)