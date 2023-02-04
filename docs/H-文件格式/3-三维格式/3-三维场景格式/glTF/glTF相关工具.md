## 查看器
|  | 说明 |
| --- | --- |
| [gltf-viewer.donmccurdy](https://gltf-viewer.donmccurdy.com/) | 网页查看器 |
| [clay-viewer](https://github.com/pissang/clay-viewer#converter) |  |
| vscode gltf tools插件 | 打开gltf文件 > alt+G打开预览 |

## 格式转换

| | 格式转换 | 说明 |
| --- | --- | --- |
| COLLADA |1. COLLADA2GLTF|  |
| OBJ | 1. [obj2gltf](https://github.com/CesiumGS/obj2gltf)|  |
| fbx |1. fbx2gltf|  |
| revit | 1. [Revit2glTF](https://github.com/McCulloughRT/Revit2glTF)|  |

## 读写库

| **库名** | **语言** | **说明** |
| --- | --- | --- |
| [glTF-CSharp-Loader](https://github.com/KhronosGroup/glTF-CSharp-Loader) | C# | 和`Interface.LoadModel("pathtommodel .gltf")`一样使用简单 |
| [微软glTF SDK](https://github.com/Microsoft/glTF-SDK) | C++ | |
| [TinyGLTF](https://github.com/syoyo/tinygltf) | C++11 |  |

## 优化
对模型进一步优化可以使用谷歌开源的Draco，大幅加速 3D 数据的编码、传输和解码。

gltf-pipeline：[https://github.com/CesiumGS/gltf-pipeline](https://github.com/CesiumGS/gltf-pipeline)，依赖NODE.JS，通过命令行窗口可以对模型进行优化：

1. 转换GLB：`gltf-pipeline -i NewProject.gltf -o NewProject.glb`
2. 转换Draco：`gltf-pipeline -i NewProject.gltf -o buildDraco.gltf –d`

源文件大小为6M，压缩完后为1M，压缩比接近6:1，对于管线、精装修等mesh多的情况达不到这种压缩比。