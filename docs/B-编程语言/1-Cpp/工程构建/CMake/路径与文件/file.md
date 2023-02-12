```cmake
# 递归文件
file(GLOB_RECURSE variable [RELATIVE path] 
   [FOLLOW_SYMLINKS] [globbing expressions]...)
# 非递归文件（当前目录下的文件）
file(GLOB Sources *.cpp)

# 示例
file (GLOB Sources *.cpp)
file (GLOB Headers *.h)
file (GLOB Resources *.qrc)
file (GLOB UIs *.ui)
```