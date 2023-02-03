---
title: Cpp TinyGltf
---

相关链接：

1. [TinyGltf github仓库](https://github.com/syoyo/tinygltf.git)

TinyGLTF是针对glTF2.0格式的一个读写库，它是一个基于C++11的头文件库。

1. 依赖[Niels Lohmann's json库](https://github.com/nlohmann/json)，用于解析JSON文件（也可以换成`RadpidJSON`、`devel-picojson`(C++03即可)、sajson、或者其他的Json后台）
2. 依赖`stb_image.h`、`stb_image_wrtie.h`用于图片的读写

## 示例
由于TinyGltf只是一个头文件库，所以将相关的头文件拷贝到工程中即可。

需要拷贝的文件有：`stb_image.h`、`stb_image_write.h`、`json.hpp`、`tiny_gltf.h`

| **主题** | **代码** | **说明** |
| --- | --- | --- |
| IO | [loader_example.cc](https://github.com/syoyo/tinygltf/blob/master/loader_example.cc) | 加载gltf2.0，并输出所有信息 |
|  | [saver](https://github.com/syoyo/tinygltf/blob/master/examples/saver/main.cc) | 读入gltf2.0，再输出成gltf格式 |
|  | [create_triangle_gltf](https://github.com/syoyo/tinygltf/blob/master/examples/build-gltf/create_triangle_gltf.cpp) | 创建一个三角形，并输出成gltf的格式 |
| 渲染 | [basic](https://github.com/syoyo/tinygltf/tree/master/examples/basic) | 简单的gltf查看器（没有pbr着色器） |
|  | [raytrace](https://github.com/syoyo/tinygltf/tree/master/examples/raytrace) | 简易、小巧、仅头文件的场景图形库 |
| 格式验证 | [validator](https://github.com/syoyo/tinygltf/tree/master/examples/validator) | 验证文件是否满足glTF 2.0标准 |

  
## 选项
### 加载选项
`TinyGLTF::SetPreserveimageChannels(bool onoff)`

- `true`：为加载的图像保留，与图像文件中相同的图像通道
- `false`：默认为向后兼容（图像通道被加宽到RGBA4通道）
- 此设置仅在使用内置图像加载器（STB图像加载器）时有效

### 编译选项

| 名称 | 说明 |
| - | - |
| `TINYGLTF_NOEXCEPTION` |在JSON解析中关闭C++异常。You can use -fno-exceptions or by defining the symbol JSON_NOEXCEPTION and TINYGLTF_NOEXCEPTION to fully remove C++ exception codes when compiling TinyGLTF. |
| `TINYGLTF_NO_STB_IMAGE ` |不要用`stb_image`加载图像。相反，通过设置`TinyGLTF::SetImageLoader(LoadimageDataFunction LoadImageData, void *user_data)`回调函数来自定义加载方法| 
| `TINYGLTF_NO_STB_IMAGE_WRITE` | 不要使用`stb_image_write`来写入图像。通过设置`TinyGLTF::SetImageWriter(WriteimageDataFunction WriteImageData, void *user_data)`回调函数来自定义写方法 |
| `TINYGLTF_NO_EXTERNAL_IMAGE ` |不要尝试加载外部图像文件。如果您不想在glTF解析期间加载图像文件，这个选项能帮助到你|
| `TINYGLTF_ANDROID_LOAD_FROM_ASSETS` |  从打包的应用资产中加载所有文件，而不是常规的文件系统。注意，你必须事先将一个有效的资产管理器从你的android应用程序传递给`tinygltf::asset_manager` | 
| `TINYGLTF_ENABLE_DRACO` | 打开`Draco`压缩。用户必须在项目文件中提供包含路径和链接相应的库 |
| `TINYGLTF_NO_INCLUDE_JSON` | 禁用在`ting_gltf.h`中包含`json.hpp`。因为它之前已经包含过了，或者您想在包含`tiny_gltf.h`之前使用它的自定义路径 |
| `TINYGLTF_NO_INCLUDE_RAPIDJSON` |禁止在`tiny_gltf.h`中包含`RapidJson`的头文件，因为它之前已经包含过了，或者你想在包含`tiny_gltf.h`之前使用它的自定义路径 |
| `TINYGLTF_NO_INCLUDE_STB_IMAGE` |禁用在`tiny_gltf.h`中包含`stb_image.h`，因为它之前已经包含过了，或者您想在`#include"tiny_gltf.h"`之前使用自定义路径inlucde它|
| `TINYGLTF_NO_INCLUDE_STB_IMAGE_WRITE`|Disable including stb_image_write.h from within tiny_gltf.h because it has been already included before or you want to include it using custom path before including tiny_gltf.h.|
| `TINYGLTF_USE_RAPIDJSON ` |Use RapidJSON as a JSON parser/serializer. RapidJSON files are not included in TinyGLTF repo. Please set an include path to RapidJSON if you enable this feature.|
| `TINYGLTF_USE_CPP14` |Use C++14 feature(requires C++14 compiler). This may give better performance than C++11. |

