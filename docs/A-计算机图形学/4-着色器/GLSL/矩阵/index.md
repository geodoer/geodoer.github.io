---
title: GLSL矩阵
---

## OpenGL中的矩阵
OpenGL渲染管线自带有一些矩阵变量
  

### gl_ModelViewMatrix
gl_ModelViewMatrix * gl_Vertex(模型坐标系下的坐标) = 相机坐标系

  

### gl_ModelViewProjectionMatrix
gl_ModelViewProjectionMatrix * gl_Vertex(模型坐标系下的坐标) = gl_Position(屏幕坐标)

```cpp
static const char* vert_src =
{
    "// gl2_VertexShader\n"
    "#ifdef GL_ES\n"
    "		precision highp float;\n"
    "#endif\n"
    "varying vec2 texCoord;\n"
    "varying vec4 vertexColor;\n"
    ""
    "void main(void)\n"
    "{\n"
    "		gl_Position = gl_ModelViewProjectionMatrix * gl_Vertex;\n"
    "		texCoord = gl_MultiTexCoord0.xy;\n"
    "		vertexColor = gl_Color;\n"
    "}\n"
};
```

  

### gl_NormalMatrix

  
  

### gl_ProjectionMatrix

投影矩阵

  

### ViewProjectionMatrix

  
  

## opengl3.3后

> global variable gl_ModelViewProjectionMatrix is deprecated after version 120

  

gl_ModelViewProjectionMatrix是一个内置GLSL常量，可以获取当前的视图投影变换矩阵。可是，自从opengl3.3后该常量标注为过期deprecated。

取而代之的是采用uniform的形式向着色器传递矩阵

  

1. 这样使用起来不是很不方便

2. 或者使用compatible方式继续使用老版本常量。

```glsl
uniform mat4 projMat;
uniform mat4 viewMat;
uniform mat4 modelMat;

layout (location = 0) in vec3 position;

void main()
{
  gl_Position = projMat * viewMat * modelMat * vec4(position, 1.0);
}
```


> 为什么新版本的opengl会丢弃这些好用的常量呢，类似的还有gl_ModeView、gl_Vertex、gl_NormalMatrix、gl_ModelViewMatrix、gl_ProjectionMatrix等。
> 
> 这些内置矩阵与函数对于开发简单的OpenGL程序与理解矩阵变换很有帮助。不过，一旦你的应用程序变得复杂起来，更好的选择是为每一个可移动模型对象管理自己的矩阵实现。此外，在OpenGL可编程管线（GLSL）中，你不能够使用这些内置矩阵函数。如OpenGL v3.0+、OpenGL ES v2.0+与WebGL v1.0+。你必须拥有属于自己的矩阵实现，并且向OpenGL着色语言传递该矩阵数据。

## 参考文章
1. [openGL之API学习（九十一）gl_ModelViewProjectionMatrix过期deprecated](https://blog.csdn.net/hankern/article/details/92325804)