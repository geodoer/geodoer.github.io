---
title: Premake
---

Premake是一个命令行工具，它读取用户预定义脚本，来执行构建配置任务，或者生成Visual Studio，Xcode和GNU Make的工程。

- 轻量：在工程中包含`premake5.exe`（1.33Mb）即可
- 易用与灵活：Premake使用轻量的lua脚本，lua为Premake带来易用与灵活性

## 使用

1. 将工程配置写成`*.lua`文件
2. 使用`premake.exe`运行即可生成指定工程

```
premake5.exe --file=../premake5.lua gmake
premake5.exe --file=../premake5.lua --cc=clang gmake
premake5.exe --file=../premake5.lua vs2015
premake5.exe --file=../premake5.lua vs2017
```

## 相关链接

1. [https://github.com/premake/premake-core/wiki](https://github.com/premake/premake-core/wiki)
