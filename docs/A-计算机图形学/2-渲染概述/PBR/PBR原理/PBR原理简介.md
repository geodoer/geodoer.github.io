## PBR简介

在图形学渲染中，有一个问题很棘手：光线打在三维模型上之后，在屏幕上要显示什么颜色？如何计算出这个颜色呢？PBR就是一种计算公式，它所计算出来的颜色，和真实世界非常像，这正是因为PBR公式是基于物理原理所总结出的，因此得名“基于物理的渲染”，英文名“Physically-Based Rendering”。

PBR公式里有一堆输入参数，例如基础色、金属度等等，有了这些输入参数渲染器才能根据PBR公式得出逼真的颜色。这些参数其实存储在材质当中，因此包含了PBR输入参数的材质称为“PBR材质”。

本文章将自顶向下的介绍PBR公式，在过程中简要介绍涉及的相关内容。
![](images/PBR原理简介.png)

## color
针对物体表面上的一个点$p$，场景中各种光线打过来，这一点会显示什么颜色$color$呢？

color由两个个部分贡献   
$$
color = colorInLights + colorEmissive
$$

1. $colorInLights$在场景中各个光线作用下，这一点显示出来的颜色
2. $colorEmissive$物体自己发出的光，简称自发光

### 各个光线所作用的颜色
场景中可能会有N个光源，`colorInLights`是这些光源共同作用之后的颜色。

$colorInLights = colorInLight_1 + colorInLigh_2 + ... + colorLight_n$

光源有四种类型

| 类型 | 说明 |
| - | - |
| 环境光 | 没有开灯的房间也不会是乌漆嘛黑的，因为太阳光会在地球上弹来弹去，最终进入房间。因此环境光也被称为间接光照 |
| 点光源 |  光源的能量会随着距离的增大而衰减 |
| 定向灯 | 光线拥有恒定的能量，并且不会随着距离的增大而衰减 |
| 聚光灯 | 光源的能量集中打在一个地方 |

其中

1. 在计算机当中，环境光一般用一个环境贴图来模拟，后续会再展开介绍。
2. 而剩下的光源（点光源、定向灯、聚光灯）都是用BRDF公式进行计算，只不过它们到达物体表面的能量不同。

伪代码

```cpp
vec4 evaluateLights(const PbrMaterial material) 
{
	PixelParams pixel = getPixelParams(material); //描述点p（物体表面的一个点）的一些信息量

	vec3 colorInLights = vec3(0.0);

	colorInLights += evaluateIBL(material); //环境光

	//遍历所有光源（除了环境光）
	for(Light light : material.Lights)
	{
		//计算光线在点p处贡献的颜色
		colorInLights += evaluateBRDF(pixel, light, material);
	}

#if defined(BLEND_MODE_FADE) && !defined(SHADING_MODEL_UNLIT)
	colorInLights *= material.baseColor.a;
#endif

	float alpha = computeDiffuseAlpha(material.baseColor.a);
	return vec4(colorInLights, alpha);
}
```

### 自发光的颜色
todo: 自发光

## BRDF公式
BRDF公式帮助我们计算出：**一条光线** 打到物体表面一点上，应该成什么颜色。

- BRDF，Bidirectional Reflective Distribution Function，双向反射分布函数

### 镜面反射与漫反射
根据物理原理，我们知道当一束入射光（incident light）打在一个表面上时，将分成两部分：镜面反射（specular reflection）和漫反射（diffuse reflection）。

![](images/漫反射与镜面反射.jpg)

它们还有很多别名，归纳如下

- 镜面反射：specular reflection、高光反射
- 漫反射：diffuse reflection、折射+散射

#### 镜面反射
根据物理原理，当一束光碰撞到平面上时，它会沿着与平面法向量的镜面方向，反射出去，而不进入平面。



直接反射，主要分布在入射光相当于平面法向量的镜像方向，与粗糙度相关。

#### 漫反射
漫反射：折射+散射，方向随机，可认为各个方向能量相同。


#### 能量守恒
根据热力学第一定律（ **能量守恒** 定律），我们有以下推论

1. 记入射光的总能量为1（单位1）
2. 假设镜面反射部分占总能量的$k_s$（如$k_s=0.8$，即代表80%的能量参与镜面反射作用）
3. 根据能量守恒定律，参与漫反射部分的能量即占总能量的$k_d = 1-k_s$（即$k_d=0.2$）

[Tips] 需强调一点的是，只是20%的能量 **参与** 漫反射作用，但实际上反射出来多少能量，是不一定的。这要根据物体的材质，如果是金属材质，它反射出来的能量几乎为0，因为光线进入物体之后，会被金属完全吸收。

```cpp
vec3 evaluateBRDF(const PixelParams pixel, const Light light, const PbrMaterial material)
{
	//计算一些参数，如NoV, NoL, NoH, LoH等，后续要用
	//..

	

	vec3 specular = evaluateSpecular(...);   //镜面反射的值


	
}
```


### 镜面反射部分
```cpp
vec3 evaluateSpecular(...)
{
	//计算光线达到点p处的能量
	float radiance = evaluateRadiance(light, pixel); 
	
	float NDF = evaluateDistributionGGX(...);  //法线方程  
	float G   = evaluateGeometrySmith(...);    //几何函数
	vec3 F    = evaluateFresnelSchlick(...);   //菲涅尔函数

	float denominator = ...;                  //根据一些参数计算出一个常量
	vec3 specular = NDF * G * F / denominator;
	
	return specular;
}
```

### 漫反射部分
```

```


### BRDF的种类
其实，科学家们总结出了很多种BRDF，每一种都能近似得出“物体表面对于光的反应”，都很逼真。但是当前几乎很多实时渲染管线使用的都是Cook-Torrance BRDF模型，上文介绍的也是这一种。

## 环境光

## 总结
在上文当中，频繁看到物理学的身影，它正是为了让渲染结果和现实世界同频所引入的。
