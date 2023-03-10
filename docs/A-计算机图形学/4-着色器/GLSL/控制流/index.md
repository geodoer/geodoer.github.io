---
title: GLSL控制流
---

## 控制流
### 条件判断
```glsl
color = unlitColor;
if (numLights > 0){
    color = litColor;
} else{
    color = unlitColor;
}
```

### 循环
和C/C++相似，提供for、while、do/while循环方式。提供continue、break控制循环
```glsl
for (l = 0; l < numLights; l++) {
  if (!lightExists[l]) continue;
  color += light[l];
}

while (i < num){
    sum += color[i];
    i++;
}

do{
    color += light[lightNum];
    lightNum--;
}while (lightNum > 0)
```

### discard
片段着色器中有一种特殊的控制流。
使用discard会退出片段着色器，不执行后面的片段着色操作。片段也不会写入缓冲区
```glsl
if (color.a < 0.9)
　　discard;
```

## 参考文章

1. [OpenGL3:高级篇 GLSL](https://www.cnblogs.com/k5bg/p/12552115.html)
