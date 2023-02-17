```cmake
# 引入opencv
if(WIN32)
	add_definitions(-D_CRT_SECURE_NO_DEPRECATE)
	find_path(OpenCV_DIR OpenCVConfig.cmake PATHS 
	          $ENV{OpenCV_DIR}
			  "D:/opencv/opencv-4.7.0" #opencv的install目录
			  )
	find_package(OpenCV REQUIRED)
else()
	find_package(OpenCV REQUIRED)
endif()

# 打印
MESSAGE(STATUS ${OpenCV_LIBS})
MESSAGE(STATUS ${OpenCV_INCLUDE_DIRS})
MESSAGE(STATUS ${OpenCV_SHARED})

# 链接到target
target_link_libraries(target_name PRIVATE ${OpenCV_LIBS})
```

我们可以自己创建一个OpenCV Target，再后续链接时，只需链接OpenCV即可
```cmake
# 创建一个OpenCV Target
add_library(OpenCV INTERFACE)
target_link_libraries(OpenCV INTERFACE ${OpenCV_LIBS})

# Target链接
target_link_libraries(target_name PRIVATE OpenCV)
```
