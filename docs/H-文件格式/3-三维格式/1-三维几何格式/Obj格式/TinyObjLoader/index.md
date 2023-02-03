---
title: tinyobjloader
---

相关链接

1. [Github地址](https://github.com/tinyobjloader/tinyobjloader)
2. [使用案例：实现OBJ文件的载入](https://blog.csdn.net/qjh5606/article/details/89075014)
3. [OpenGL+TinyObjLoader加载Obj](https://www.cnblogs.com/feifanrensheng/p/9416717.html)


## 库中数据结构

### 顶点数据信息attrib_t
主要存放的是OBJ文件中所有顶点数据信息，并且OBJ中只有一份顶点数据，其他存的都是索引

```cpp
typedef struct {
  std::vector<real_t> vertices;   // 'v'  顶点位置信息
  std::vector<real_t> normals;    // 'vn' 法线信息
  std::vector<real_t> texcoords;  // 'vt' 纹理坐标信息
  std::vector<real_t> colors;     // extension: vertex colors 颜色信息
} attrib_t;
```

### 物体的一部分shape_t
shape_t表示物体的一部分

在文件中`o xxx`表示这部分的开始：存储的信息有：

```cpp
typedef struct {
  std::string name; //这部分的名称
  mesh_t mesh;    //构成这部分的顶点信息，通过使用索引的方式记录（顶点数据都在attrib_t中）
  path_t path;    //pairs of indices for lines [?]线段的索引
} shape_t;
```

【顶点信息mesh_t（索引）】

```cpp
// 索引结构支持不同的索引，如vtx/normal/texcoord.
// -1表示未使用
typedef struct {
  int vertex_index; //顶点位置信息
  int normal_index; //法线信息
  int texcoord_index; //纹理坐标信息
} index_t; //索引信息

typedef struct {
  std::vector<index_t> indices; //索引信息
  std::vector<unsigned char> num_face_vertices; //每个face的顶点数
    //3 = polygon, 4 = quad, ... Up to 255.
  std::vector<int> material_ids;                 //每个face的材料标识
  std::vector<unsigned int> smoothing_group_ids; //每个face的smoothing组
  // ID(0 = off. positive value = group id)
  std::vector<tag_t> tags;                        // SubD tag
} mesh_t;
```