---
title: GLSL变量与内置变量
---

## 变量
GLSL的变量命名方式与C语言类似

- 变量名称可以使用字母、数字以及下划线
- 不能以数字开头
- 不能以`gl_`作为前缀，这是GLSL保留的前缀，用于GLSL的内部变量
- 还有一些GLSL保留的名称，也是不允许使用的

## 内置变量
### 顶点着色器
| 名称 | 类型 | 描述 |
| --- | --- | --- |
| gl_Color | vec4 | 输入属性-表示顶点的主颜色 |
| gl_SecondaryColor | vec4 | 输入属性-表示顶点的辅助颜色 |
| gl_Normal | vec3 | 输入属性-表示顶点的法线值 |
| gl_Vertex | vec4 | 输入属性-表示物体空间（物体坐标系）的顶点位置 |
| gl_MultiTexCoordn | vec4 | 输入属性-表示顶点的第n个纹理的坐标 |
| gl_FogCoord | float | 输入属性-表示顶点的雾坐标 |
| gl_Position | vec4 | 输出属性-变换后的顶点的位置（屏幕坐标系），用于后面的固定的裁剪等操作。所有的顶点着色器都必须写出这个值。 |
| gl_ClipVertex | vec4 | 输出坐标，用于用户裁剪平面的裁剪 |
| gl_PointSize | float | 点的大小（默认值1，通常设置绘图为点绘制才有意义） |
| gl_FrontColor | vec4 | 正面的主颜色的varying输出 |
| gl_BackColor | vec4 | 背面主颜色的varying输出 |
| gl_FrontSecondaryColor | vec4 | 正面的辅助颜色的varying输出 |
| gl_BackSecondaryColor | vec4 | 背面的辅助颜色的varying输出 |
| gl_TexCoord[] | vec4 | 纹理坐标的数组varying输出 |
| gl_FogFragCoord | float | 雾坐标的varying输出 |


### 片段着色器
| 名称 | 类型 | 描述 |
| --- | --- | --- |
| gl_Color | vec4 | 包含主颜色的插值只读输入 |
| gl_SecondaryColor | vec4 | 包含辅助颜色的插值只读输入 |
| gl_TexCoord[] | vec4 | 包含纹理坐标数组的插值只读输入 |
| gl_FogFragCoord | float | 包含雾坐标的插值只读输入 |
| gl_FragCoord | vec4 | 只读输入，窗口的x,y,z和1/w |
| gl_FrontFacing | bool | 只读输入，如果是窗口正面图元的一部分，则这个值为true |
| gl_PointCoord | vec2 | 点精灵的二维空间坐标范围在(0.0, 0.0)到(1.0, 1.0)之间，仅用于点图元和点精灵开启的情况下。 |
| gl_FragData[] | vec4 | 使用glDrawBuffers输出的数据数组。不能与gl_FragColor结合使用。 |
| gl_FragColor | vec4 | 输出的颜色用于随后的像素操作 |
| gl_FragDepth | float | 输出的深度用于随后的像素操作，如果这个值没有被写，则使用固定功能管线的深度值代替 |

## 参考文章

1. [OpenGL3:高级篇 GLSL](https://www.cnblogs.com/k5bg/p/12552115.html)
