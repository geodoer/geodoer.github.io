可参考源代码中的loader_exmaple.cc

1. 加入源代码：将tiny_obj_loader.h放入工程
2. include：
```cpp
#define TINYOBJLOADER_IMPLEMENTATION
#include "tiny_obj_loader.h"
```

3. 读取并存储数据
```cpp
bool Object::make_mesh_and_material_by_obj(const char* filename,
                                           const char* basepath,
                                           bool triangulate){
  std::cout << "Loading " << filename << std::endl;

  tinyobj::attrib_t attrib; // 所有的数据放在这里

  std::vector<tinyobj::shape_t> shapes;

  // 一个shape,表示一个部分,
  // 其中主要存的是索引坐标mesh_t类,放在indices中
  std::vector<tinyobj::material_t> materials;
  std::string warn;
  std::string err;
  
  bool ret = tinyobj::LoadObj(&attrib, &shapes, &materials, &warn, &err, filename,
    basepath, triangulate);

  // 接下里就是从上面的属性中取值了
  if (!warn.empty()) {
    std::cout << "WARN: " << warn << std::endl;
  }
  if (!err.empty()) {
    std::cerr << "ERR: " << err << std::endl;
  }
  if (!ret) {
    printf("Failed to load/parse .obj.\n");
    return false;
  }
  std::cout << "# Loading Complete #"<< std::endl;
  //PrintInfo(attrib, shapes, materials);
  return true;
}
```