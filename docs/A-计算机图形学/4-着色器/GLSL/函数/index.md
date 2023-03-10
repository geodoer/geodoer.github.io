---
title: GLSL函数
---

- GLSL中的函数，必须是在全局范围定义和声明的。不能在函数定义中声明或定义函数。
- 函数必须有返回类型，参数是可选的。
- 参数的修饰符(in, out, inout, const等）是可选的，结构体和数组也可以作为函数的参数。如果是数组作为函数的参数，则必须制定其大小。在调用传参时，只传数组名就可以了。
- GLSL的函数是支持重载的。函数可以同名但其参数类型或者参数个数不同即可

## 主函数
在每个shader中必须有一个main函数
main函数中的void参数是可选的，但是返回值必须是void
```glsl
void main(void){
 ...
}
```

## texture
相关链接

1. [官方文档：texture函数](https://www.opengl.org/sdk/docs/man/html/texture.xhtml)

## mix
mix函数用于颜色插值，函数原型为：
```c
genType mix(genType x, genType y, genType a);
```

最终值的计算方法为：

$$
x×(1−a)+y×a
$$

## 参考文章

1. [OpenGL3:高级篇 GLSL](https://www.cnblogs.com/k5bg/p/12552115.html)
