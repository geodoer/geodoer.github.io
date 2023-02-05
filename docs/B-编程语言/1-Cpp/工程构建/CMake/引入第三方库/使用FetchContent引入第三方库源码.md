---
tags:
    - cmake
    - FetchContent
---

在cmake3.14版本之后，可以使用`FetchContent`相关语法下载并引入第三方库源码。

优点：无需事先将源代码下载到本地，cmake会自动下载源码

缺点

- 若第三方库源码很多，或依赖外网，下载会很慢。例如libigl的下载就很慢（依赖太多）
- 第三方库需支持CMake


## 示例：libigl
```
.
├── cmake
│   └── libigl.cmake
├── CMakeLists.txt
└── src
	├── ...
    └── main.cpp
```

文件：libigl.cmake
```cmake
if(TARGET igl::core)
    return() #如果已经存在，则退出
endif()

# 引入FetchContent相关函数
include(FetchContent)

# 通过FetchContent在网上下载源代码
FetchContent_Declare(
    libigl  #TARGET名称
    GIT_REPOSITORY https://github.com/libigl/libigl.git #网址
    GIT_TAG v2.4.0 #tag
)

# 确保下载成功，并添加到cmake
FetchContent_MakeAvailable(libigl)
```

文件：CMakeLists.txt
```cmake
cmake_minimum_required(VERSION 3.16)
project(surface_subdivision)

# 
# 将cmake目录加入搜索路径
#
list(PREPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

# 引入Libigl模块
option(LIBIGL_GLFW                "Build target igl::glfw"                ON)
option(LIBIGL_IMGUI               "Build target igl::imgui"               ON)
option(LIBIGL_COPYLEFT_CGAL       "Build target igl_copyleft::cgal"       ON)
include(libigl) #引入libigl.cmake文件

#
# 工程代码
# 
set(Project_Include_Dir "${CMAKE_CURRENT_SOURCE_DIR}/include")
include_directories(${Project_Include_Dir})
file(GLOB_RECURSE Project_Header_Files ${Project_Include_Dir}/*.h ${Project_Include_Dir}/*.hpp)
set(Project_Src_Dir "${CMAKE_CURRENT_SOURCE_DIR}/src")

file(GLOB_RECURSE SRC_FILES ${Project_Src_Dir}/*.cpp ${Project_Src_Dir}/*.h)
add_executable(${PROJECT_NAME} ${SRC_FILES} ${Project_Header_Files})
target_link_libraries(${PROJECT_NAME} PUBLIC igl::glfw igl::imgui igl_copyleft::cgal)
  #直接链接即可

#
# VS工程目录
#

# 代码分组
source_group(TREE ${CMAKE_SOURCE_DIR} FILES ${SRC_FILES})
source_group(TREE ${CMAKE_SOURCE_DIR} FILES ${Project_Header_Files})
```


## 参考文章
1. [https://cmake.org/cmake/help/latest/module/FetchContent.html#id1](https://cmake.org/cmake/help/latest/module/FetchContent.html#id1)
