---
title: FbxSDK C++示例
---

## 官方示例
示例在fbx sdk的安装目录下，使用cmake一下就能编过。

问题：找不到对应的lib

1. 将``C:\Program Files\Autodesk\FBX\FBX SDK\2020.0.1\lib\vs2017`的内容拷贝到目录`C:\Program Files\Autodesk\FBX\FBX SDK\2020.0.1\lib``中

示例说明：
```
samples
- ExportScene03
	- 继承FbxMesh、FbxObject的类，创建属于自己的MyKFbxMesh和MyFbxObject
		- 注册自己的类；
		- 属性的创建
	- 创建带有纹理的cube
		- 法向量：添加法向量、一个控制点一个法向量
		- 创建纹理
	- 创建带材质的立方体
	- 创建三角形
- ExportScene04 相机
- ExportScene05 说明了属性层次结构的构造
```

