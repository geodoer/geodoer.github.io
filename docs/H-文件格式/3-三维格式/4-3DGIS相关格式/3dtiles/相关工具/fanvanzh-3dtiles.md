源代码：[https://github.com/fanvanzh/3dtiles](https://github.com/fanvanzh/3dtiles)

项目说明：3dtiles格式转换器

1. 由C++子工程提供基础的函数，最终由rust组装并编写CUI
2. C++代码依赖osg、gdal、nlohmann json、stb_image与tiny_gltf
3. 作者不是专业搞三维GIS，因偶尔需要展示3D Tiles，一时找不到工具，于是有了这个工具

使用记录

1. 打开`vs/vs/sln`，切换至release、x64即可编译成功，生成了3dtile.dll

## 源码阅读
src目录，包含

- C++代码
- rust代码

 C++代码

- `extern.h` 

## 调试
相关文档：[How-to-build](https://github.com/fanvanzh/3dtiles/wiki/How-to-build)

1. 下载pre-build版本：[https://github.com/fanvanzh/3dtiles/releases/download/v0.4/3dtile.zip](https://github.com/fanvanzh/3dtiles/releases/download/v0.4/3dtile.zip)
2. 将pre-build版本解压
3. 打开`vs/vs.sln`，切换至debug与x64
4. 右键3dtile > 属性 > 调试 > 命令 > 输入`E:\web3d\3dtiles\vs\x64\Debug\3dtile.exe`
5. 打断点，运行即可

## 编译


