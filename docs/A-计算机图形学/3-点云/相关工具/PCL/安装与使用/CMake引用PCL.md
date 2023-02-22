## CMake引用PCL
### 引入PCL
```cpp
#
# 引入PCL
#
if(WIN32)
	add_definitions(-D_CRT_SECURE_NO_DEPRECATE)
	find_path(PCL_DIR PCLConfig.cmake PATHS 
		$ENV{PCL_DIR}
		"D:/install/PCL 1.13.0"
	)
endif()
find_package(PCL REQUIRED)

MESSAGE(STATUS "[PCL_INCLUDE_DIRS] ${PCL_INCLUDE_DIRS}")
MESSAGE(STATUS "[PCL_LIBRARY_DIRS] ${PCL_LIBRARY_DIRS}")
MESSAGE(STATUS "[PCL_LIBRARIES] ${PCL_LIBRARIES}")
MESSAGE(STATUS "[PCL_DEFINITIONS] ${PCL_DEFINITIONS}")
```

### 链接
#### 方式一：使用PCL自带的变量
```cpp
add_executable(pcd_write_test pcd_write.cpp)

target_include_directories(pcd_write_test PRIVATE ${PCL_INCLUDE_DIRS})
target_link_directories(pcd_write_test PRIVATE ${PCL_LIBRARY_DIRS})
target_link_libraries(pcd_write_test PRIVATE ${PCL_LIBRARIES})
# add_compile_definitions(PCL PRIVATE ${PCL_DEFINITIONS})
```

#### 方法二：自己创建Target，方便链接
```cmake
# 自己创建一个PCL Target
add_library(PCL INTERFACE)
target_include_directories(PCL INTERFACE ${PCL_INCLUDE_DIRS})
target_link_directories(PCL INTERFACE ${PCL_LIBRARY_DIRS})
target_link_libraries(PCL INTERFACE ${PCL_LIBRARIES})
# add_compile_definitions(PCL INTERFACE ${PCL_DEFINITIONS})

# 子工程直接引用PCL即可
add_executable(pcd_write_test pcd_write.cpp)
target_link_libraries(pcd_write_test PRIVATE PCL)
```

## 对于PCL的依赖库
如果我们还需要使用到PCL的一些依赖库，如boost等。  
这就需要我们手动链接它们，因为`${PCL_LIBRARIES}`变量中并不包含`boost`。

```cmake
find_package(PCL REQUIRED)

# 在PCL find_package之后即可，因为它会将依赖库的相关内容加入进来
find_package(Boost REQUIRED COMPONENTS thread)

# 
# 链接 
#
add_executable(pcd_write_test pcd_write.cpp)
target_link_libraries(pcd_write_test PRIVATE PCL 
	Boost::thread #手动链接
)
```