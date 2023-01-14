---
tags:
    - cmake
    - find_path
---


## 示例

```
find_path(GLEW_INCLUDE_DIR
  NAME GL/glew.h
  PATHS ${CMAKE_CURRENT_LIST_DIR}
)

set(GLEW_INCLUDE_DIRS ${GLEW_INCLUDE_DIR})
# message(STATUS ${GLEW_INCLUDE_DIRS})

if(CMAKE_SIZEOF_VOID_P EQUAL 8)
  set(_arch "x64")
else()
  set(_arch "Win32")
endif()
# message(STATUS ${_arch})

find_library(GLEW_SHARED_LIBRARY_RELEASE
             NAMES GLEW glew glew32
             PATH_SUFFIXES lib lib64 libx32 lib/Release/${_arch}
             PATHS ${CMAKE_CURRENT_LIST_DIR})
message(STATUS ${GLEW_SHARED_LIBRARY_RELEASE})

find_library(GLEW_SHARED_LIBRARY_DEBUG
             NAMES GLEWd glewd glew32d
             PATH_SUFFIXES lib lib64
             PATHS ${CMAKE_CURRENT_LIST_DIR})
message(STATUS ${GLEW_SHARED_LIBRARY_DEBUG})

find_library(GLEW_STATIC_LIBRARY_RELEASE
            NAMES GLEW glew glew32s
            PATH_SUFFIXES lib lib64 libx32 lib/Release/${_arch}
            PATHS ${CMAKE_CURRENT_LIST_DIR})
message(STATUS ${GLEW_STATIC_LIBRARY_RELEASE})

find_library(GLEW_STATIC_LIBRARY_DEBUG
             NAMES GLEWds glewds glew32ds
             PATH_SUFFIXES lib lib64
             PATHS ENV ${GLEW_SHARED_LIBRARY_DEBUG})
message(STATUS ${GLEW_STATIC_LIBRARY_DEBUG})

if(NOT GLEW_USE_STATIC_LIBS)
  set(GLEW_LIBRARIES ${GLEW_SHARED_LIBRARY})
  message(STATUS ${GLEW_LIBRARIES})
else()
  set(GLEW_LIBRARIES ${GLEW_STATIC_LIBRARY})
  message(STATUS "STATIC"  ${GLEW_LIBRARIES})
endif()
```