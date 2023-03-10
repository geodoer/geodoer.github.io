---
title: GLSL类型
---

## 基本类型
GLSL引入了一些在着色器中经常用到的类型作为基本类型

| 类型 | 描述 |
| --- | --- |
| void | 跟C语言的void类似，表示空类型。作为函数的返回类型，表示这个函数不返回值。 |
| bool | 布尔类型，可以是true 和false，以及可以产生布尔型的表达式。 |
| int | 整型 代表至少包含16位的有符号的整数。可以是十进制的，十六进制的，八进制的。 |
| float | 浮点型 |
| bvec2 | 包含2个布尔成分的向量 |
| bvec3 | 包含3个布尔成分的向量 |
| bvec4 | 包含4个布尔成分的向量 |
| ivec2 | 包含2个整型成分的向量 |
| ivec3 | 包含3个整型成分的向量 |
| ivec4 | 包含4个整型成分的向量 |
| mat2 或者 mat2x2 | 2x2的浮点数矩阵类型 |
| mat3或者mat3x3 | 3x3的浮点数矩阵类型 |
| mat4x4 | 4x4的浮点矩阵 |
| mat2x3 | 2列3行的浮点矩阵（OpenGL的矩阵是列主顺序的） |
| mat2x4 | 2列4行的浮点矩阵 |
| mat3x2 | 3列2行的浮点矩阵 |
| mat3x4 | 3列4行的浮点矩阵 |
| mat4x2 | 4列2行的浮点矩阵 |
| mat4x3 | 4列3行的浮点矩阵 |
| sampler1D | 用于内建的纹理函数中引用指定的1D纹理的句柄。只可以作为一致变量或者函数参数使用 |
| sampler2D | 二维纹理句柄 |
| sampler3D | 三维纹理句柄 |
| samplerCube | cube map纹理句柄 |
| sampler1DShadow | 一维深度纹理句柄 |
| sampler2DShadow | 二维深度纹理句柄 |


## 结构体
```glsl
struct surface {
  float indexOfRefraction;
  vec3 color;float turbulence;
} mySurface;

surface secondeSurface;
```

## 数组
GLSL只有一维数组

- 数组下标从0开始，合理的范围是[0, size-1]
- 与C语言一样，如果数组访问越界，那行为是未定义的。如果着色器编译器在编译期时知道数组访问越界，则会提示编译失败
```glsl
surface mySurfaces[];
vec4 lightPositions[8];
vec4 lightPos[] = lightPositions;
const int numSurfaces = 5;
surface myFiveSurfaces[numSurfaces];
float[5] values;

vec4 myColor, ambient, diffuse[6], specular[6];
myColor = ambient + diffuse[4] + specular[4];
```

## 参考文章

1. [OpenGL3:高级篇 GLSL](https://www.cnblogs.com/k5bg/p/12552115.html)
