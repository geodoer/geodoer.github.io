参考链接：

1.  [glTF教程_MinimalGltfFile](https://github.com/fangcun010/glTF-Tutorials/blob/master/gltfTutorial/gltfTutorial_003_MinimalGltfFile.md)；[中文翻译](https://zhuanlan.zhihu.com/p/65267849)

本文介绍一个最小的glTF文件，它描述了一个简单的三角形。

![](images/最小的glTF文件（简单三角形）.png)


## Model（glTF文件）

此示例只有一个glTF文件。

```json
{
  "scenes" : [
    {
      "nodes" : [ 0 ]
    }
  ],
  
  "nodes" : [
    {
      "mesh" : 0
    }
  ],
  
  "meshes" : [
    {
      "primitives" : [ {
        "attributes" : {
          "POSITION" : 1
        },
        "indices" : 0
      } ]
    }
  ],

  "buffers" : [
    {
      "uri" : "data:application/octet-stream;base64,AAABAAIAAAAAAAAAAAAAAAAAAAAAAIA/AAAAAAAAAAAAAAAAAACAPwAAAAA=",
      "byteLength" : 44
    }
  ],
  "bufferViews" : [
    {
      "buffer" : 0,
      "byteOffset" : 0,
      "byteLength" : 6,
      "target" : 34963
    },
    {
      "buffer" : 0,
      "byteOffset" : 8,
      "byteLength" : 36,
      "target" : 34962
    }
  ],
  "accessors" : [
    {
      "bufferView" : 0,
      "byteOffset" : 0,
      "componentType" : 5123,
      "count" : 3,
      "type" : "SCALAR",
      "max" : [ 2 ],
      "min" : [ 0 ]
    },
    {
      "bufferView" : 1,
      "byteOffset" : 0,
      "componentType" : 5126,
      "count" : 3,
      "type" : "VEC3",
      "max" : [ 1.0, 1.0, 0.0 ],
      "min" : [ 0.0, 0.0, 0.0 ]
    }
  ],
  
  "asset" : {
    "version" : "2.0"
  }
}
```

将这个glTF文件读入之后，你就能拿到一个Model对象，即一个glTF格式对应一个Model

在Model下，全部数据按类型存储在一个一维数组中，而别的地方使用**下标**来索引到指定数据。

```cpp
class Model{
  std::vector<Accessor> accessors;
  std::vector<Animation> animations;
  std::vector<Buffer> buffers;
  std::vector<BufferView> bufferViews;
  std::vector<Material> materials;
  std::vector<Mesh> meshes;
  std::vector<Node> nodes;
  std::vector<Texture> textures;
  std::vector<Image> images;
  std::vector<Skin> skins;
  std::vector<Sampler> samplers;
  std::vector<Camera> cameras;
  std::vector<Scene> scenes;
  std::vector<Light> lights;
};
```

除了三维方面的信息，Model下还保存了一些属性信息

```cpp
class Model{
  int defaultScene = -1;	//默认的场景（这个值是场景的下标）
  std::vector<std::string> extensionsUsed;
  std::vector<std::string> extensionsRequired;

  Asset asset;

  Value extras;
  ExtensionMap extensions;

  // Filled when SetStoreOriginalJSONForExtrasAndExtensions is enabled.
  std::string extras_json_string;
  std::string extensions_json_string;
}
```

其中`extras`、`extensions`与`extras_json_string`、`extensions_json_string`在其他概念中也有，后面将不赘述了。

### Asset

其中Asset表示此glTF的元数据，包括版本、创建者、版权等等。

```cpp
struct Asset {
  std::string version = "2.0";  // required
  std::string generator;
  std::string minVersion;
  std::string copyright;
};
```

## Scene与Node
```json
  "scenes" : [
    {
      "nodes" : [ 0 ]
    }
  ],
  "nodes" : [
    {
      "mesh" : 0
    }
  ],
```
glTF标准引用了场景的概念，一个Model下包含了多个Scene

一个Scene包含了多棵树（即多个根节点）

- 其中Node存储在Model中
- 而Scene中存储的是Node的下标，通过下标引用Node

```cpp
class Model{	//gltf : Model = 1:1
	std::vector<Scene> scenes;	//Model : Scene = 1:N
}

class Scene{
	std::vector<int> nodes;	//Scene : Node = 1:N，存储的是下标
}
```

Node自身是一个树状结构，一个Node包含多个子Node

- 所有的Node都存储在Model中

```cpp
class Node {
    int camera;  // the index of the camera referenced by this node
    int skin;
    int mesh;
    std::vector<int> children;				//该child在Model.nodes的下标
    std::vector<double> rotation;     // length must be 0 or 4
    std::vector<double> scale;        // length must be 0 or 3
    std::vector<double> translation;  // length must be 0 or 3
    std::vector<double> matrix;       // length must be 0 or 16
    std::vector<double> weights;  		// The weights of the instantiated Morph Target
};
```

## buffer、bufferView与accessor
glTF将实际的几何信息（坐标点的xyz，图元索引）打包存储在了外部文件当中，以此方便做文件传输与压缩存储。

但是，数据存储在外部文件中，程序很不好访问。因此，glTF定义了accessor相关概念，帮助我们从外部二进制文件中访问到我们想要的数据。

### buffer
buffer对象描述了一个没有任何结构和层次意义的数据块
它包含了一个uri属性

1. 这个uri可以引用外部的文件

```cpp
struct Buffer {
  std::string uri;	//<- 外部文件
    
  std::vector<unsigned char> data;	//<- 将外部文件读入后，存储在data中
};
```

2. 如果数据量小，或者你愿意，也可以将数据直接存储在uri中。比如，此uri中存储了44字节的buffer数据，其直接存储在了json文件中，并没有存储在外部
```json
  "buffers" : [
    {
      "uri" : "data:application/octet-stream;base64,AAABAAIAAAAAAAAAAAAAAAAAAAAAAIA/AAAAAAAAAAAAAAAAAACAPwAAAAA=",
      "byteLength" : 44
    }
  ],
```

### bufferView
bufferView类似于buffer中的一个数据块，它引用了一个buffer中的部分数据
```cpp
struct BufferView {
  int buffer{-1};        // buffer的下标
  size_t byteOffset{0};  // 位移量
  size_t byteLength{0};  // 长度
  size_t byteStride{0};  // minimum 4, maximum 252 (multiple of 4), default 0 =
                         // understood to be tightly packed
  int target{0};  // ["ARRAY_BUFFER", "ELEMENT_ARRAY_BUFFER"] for vertex indices
                  // or atttribs. Could be 0 for other data
  bool dracoDecoded{false}; //true：已被draco解码
};
```

比如：`Model.bufferViews`

- 所有的bufferView都存储在Model下
- 此Model包含两个bufferViews
```cpp
  "bufferViews" : [ //此Model中包含了两个bufferView
    {	//第一个bufferView。[0, 0+6)
      "buffer" : 0,			//引用了Model中的第一个buffer对象
      "byteOffset" : 0,		//数据的开头
      "byteLength" : 6,		//数据的长度
      "target" : 34963
    },
    {	//第二个bufferView。[8, 8+36)
      "buffer" : 0,			//引用了Model中的第二个buffer对象
      "byteOffset" : 8,		//数据的开头
      "byteLength" : 36,	//数据的长度
      "target" : 34962
    }
  ],
```

### Accessors
Accessor对象描述了bufferView的含义，bufferView的构件类型、数据类型等等。

此示例中，有两个accessors对象
```json
  "accessors" : [
    {	//顶点的索引数据
      "bufferView" : 0,	//数据块
      "byteOffset" : 0,
      "componentType" : 5123,
      "count" : 3,
      "type" : "SCALAR",
      "max" : [ 2 ],
      "min" : [ 0 ]
    },
    { //顶点位置数据
      "bufferView" : 1,	//数据块
      "byteOffset" : 0,
      "componentType" : 5126,
      "count" : 3,
      "type" : "VEC3",
      "max" : [ 1.0, 1.0, 0.0 ],
      "min" : [ 0.0, 0.0, 0.0 ]
    }
  ],
```

## Mesh与Primitive
```json
  "meshes" : [
    {
      "primitives" : [ {
        "attributes" : {
          "POSITION" : 1
        },
        "indices" : 0
      } ]
    }
  ],
```

Mesh描述三维对象的几何信息

```cpp
struct Mesh {
  std::vector<Primitive> primitives;
  std::vector<double> weights;  // weights to be applied to the Morph Targets
};
```

Mesh中包含了多个Primitive，每个Primitive对象描述了一部分mesh对象的几何数据

- Primitive意为图元，但图元本质上应该为Point、Edge、Face（三角形、四边形、多边形等）
- 但glTF中的Primitive貌似不是图元的意思，实质上是Mesh中一部分网格，可能包含多个Face

```cpp
struct Primitive {
  std::map<std::string, int> attributes;
    //key:属性类型
  	//value:访问器的索引
  int material;  
    //此Primitive的材质（也是存的索引）
  int indices;
    //索引集。存的是访问器里的下标
  int mode;
    //模式，形式犹如：TINYGLTF_MODE_***
  std::vector<std::map<std::string, int> > targets;  
  // array of morph targets,
  // where each target is a dict with attribues in ["POSITION, "NORMAL",
  // "TANGENT"] pointing
  // to their corresponding accessors
};
```

可以看到，Primitive中存储了accessor的索引，通过索引使用了几何数据
当需要使用几何数据进行渲染时，渲染程序可以通过这个引用关系，找到对应的几何数据，进行渲染
