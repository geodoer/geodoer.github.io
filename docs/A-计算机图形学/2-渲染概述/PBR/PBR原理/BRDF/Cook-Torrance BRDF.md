Cook-Torrance BRDF兼有漫反射和镜面反射两个部分：

$$
f_r = k_d f_{lambert} +  k_s f_{cook-torrance}
$$

1. 【$k_d$】入射光线中，被折射部分能量的比率
2. 【$k_s$】反射光线中，被反射部分的比率
3. $k_d$和$k_s$一般由艺术家设置，不同的材质，这两个参数的值不同

## Lambertian漫反射

BRDF加号左侧$f_{lambert}$表示的是漫反射部分，在Cook-Torrance BRDF中使用Lambertian漫反射。
具体可参见：《Lambertian漫反射》

## 镜面反射部分
Cook-Torrance BRDF的镜面反射

1. 分子包含了三个函数，$D$、$F$、$G$分别代表着一种类型的函数，它们都是用来估计一些物理参数的（具体可查阅《微平面模型》）
2. 分母部分还有一个标准化因子

$$
f_{cook-torrance} = \frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)}
$$


```cpp
vec3 evaluateSpecular(...)
{
	//计算光线达到点p处的能量
	float radiance = evaluateRadiance(light, pixel); 
	
	float NDF = evaluateDistributionGGX(...);  //法线方程  
	float G   = evaluateGeometrySmith(...);    //几何函数
	vec3 F    = evaluateFresnelSchlick(...);   //菲涅尔函数

	float denominator = ...;  //根据一些参数计算出一个常量
	vec3 specular = NDF * G * F / denominator;
	
	return specular;
}
```
