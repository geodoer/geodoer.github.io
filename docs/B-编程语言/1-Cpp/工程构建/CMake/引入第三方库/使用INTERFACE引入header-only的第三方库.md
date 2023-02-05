---
tags:
    - cmake
    - INTERFACE
---

对于header-only（只有头文件，无源文件）的第三方库，可以使用`INTERFACE`来创建TARGET，以此方便在工程中使用。

## 示例：catch2
> 例子链接[GitHub - ttroy50/cmake-examples: Useful CMake Examples](https://github.com/ttroy50/cmake-examples)中，05-unit-testing\catch2-vendored

第一步：将catch2放入工程目录当中
```
.
├── 3rd_party
│   └── catch2
│       ├── catch2
│       │   └── catch.hpp
│       └── CMakeLists.txt
├── CMakeLists.txt
└── src
	├── ...
    └── example.cpp
```

第二步：为catch2编写`CMakeLists.txt`
- 使用`INTERFACE`创建Target，方便工程中其他Target链接
```cmake
#File: .\3rd_party\catch2\CMakeLists.txt
cmake_minimum_required(VERSION 3.0)

project(catch2)

# Prepare "Catch2" library for other executables
add_library(Catch2 INTERFACE)
add_library(Catch2::Test ALIAS Catch2)
target_include_directories(Catch2 INTERFACE ${CMAKE_CURRENT_SOURCE_DIR})
```

第三步：在顶部的CMake文件中加入此模块
```cmake
#File: .\CMakeLists.txt
cmake_minimum_required(VERSION 3.5)
project (catch2_unit_test)

set(CMAKE_CXX_STANDARD 11)

# add the CMakeFile that defines catch2
add_subdirectory(3rd_party/catch2)

# Add an library for the example classes
add_library(example_unit_test
    example.cpp
)

#############################################
# Unit tests

# enable CTest testing
enable_testing()

# Add a testing executable
add_executable(unit_tests unit_tests.cpp)

target_link_libraries(unit_tests
    example_unit_test
    Catch2::Test
)

add_test(test_all unit_tests)
```
