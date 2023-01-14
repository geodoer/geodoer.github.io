---
tags:
    - cmake
    - GUI
    - CACHE
    - mark as advanced
---

## CACHE变量
CMake中有两种变量

1. Cache Variables：变量被显示申明成CACHE变量，它就会显示在CMake GUI中。如`set(MY_CACHE_VAL "666" CACHE STRING INTERNAL)`，相当于全局变量
2. Normal Variables：没有被申明成CACHE的变量，都是Normal变量。如`set(MY_VAL "666")`，作用域只在CMakeLists.txt文件当中

Cache变量的定义：
```cmake
set(<variable> <value>... CACHE <type> <docstring> [FORCE])
```

参数说明：
1. TYPE
```
FILEPATH = File chooser dialog.
PATH     = Directory chooser dialog.
STRING   = Arbitrary string.
BOOL     = Boolean ON/OFF checkbox.
INTERNAL = No GUI entry (used for persistent variables).
```
2. docstring：必须是字符串，作为变量概要说明
3. FORCE：强制选项，强制修改变量值

示例
```cmake
set(GLEW_USE_STATIC_LIBS ON CACHE BOOL "" FORCE)
```

## 高级选项
```cmake
# 将LibHello_INCLUDE_DIR设置为高级选项
set(LibHello_INCLUDE_DIR ${LibHello_INCLUDE_DIR} CACHE)	#将该变量设置成cache变量
mark_as_advanced(LibHello_INCLUDE_DIR)  #只有cache变量才能被设置为高级选项
```

## 参考链接
1.  [CMake官方文档——set](https://cmake.org/cmake/help/latest/command/set.html)
2.  [cmake cache变量](https://blog.csdn.net/weixin_39732534/article/details/110658282)