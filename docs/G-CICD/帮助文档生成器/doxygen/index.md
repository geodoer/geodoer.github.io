---
title: doxygen
---

Doxygen是一款文档生成工具，它可以从代码中提取出相应的文档，并组织，输出成各种漂亮的文档（如HTML，PDF，RTF等）

-   适用于[C++](https://zh.wikipedia.org/wiki/C%2B%2B)、[C](https://zh.wikipedia.org/wiki/C)、[Java](https://zh.wikipedia.org/wiki/Java)、[Objective-C](https://zh.wikipedia.org/wiki/Objective-C)、[Python](https://zh.wikipedia.org/wiki/Python)、[IDL](https://zh.wikipedia.org/wiki/%E6%8E%A5%E5%8F%A3%E6%8F%8F%E8%BF%B0%E8%AF%AD%E8%A8%80)（[CORBA](https://zh.wikipedia.org/wiki/CORBA)和Microsoft flavors）、[Fortran](https://zh.wikipedia.org/wiki/Fortran)、[VHDL](https://zh.wikipedia.org/wiki/VHDL)、[PHP](https://zh.wikipedia.org/wiki/PHP)、[C#](https://zh.wikipedia.org/wiki/C_Sharp)和[D语言](https://zh.wikipedia.org/wiki/D%E8%AA%9E%E8%A8%80)的文档生成器
-   它可以在大多数[类Unix](https://zh.wikipedia.org/wiki/%E7%B1%BBUnix)[操作系统](https://zh.wikipedia.org/wiki/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%B5%B1)、[macOS](https://zh.wikipedia.org/wiki/MacOS)以及[Microsoft Windows](https://zh.wikipedia.org/wiki/Microsoft_Windows)上运行。

## 相关链接

1.  [官方文档](https://www.doxygen.nl/index.html)
2.  [Doxygen使用教程](https://zhuanlan.zhihu.com/p/100223113)
3.  [Doxygen + graphviz + Windows Help Workshop生成函数调用图和chm文件](https://blog.csdn.net/imgcl/article/details/79480350)

## 安装

1.  安装 [Doxygen(Windows)](https://doxygen.nl/files/doxygen-1.9.1-setup.exe) 
2.  安装 [graphviz(Windows)](https://graphviz.org/download/)：graphviz 是一个由AT&T实验室启动的开源工具包，用于绘制DOT语言脚本描述的图形。Doxygen 使用 graphviz 自动生成类之间和文件之间的调用关系图，如不需要此功能可不安装该工具包。
3.  安装 [Windows Help Workshop](https://www.helpandmanual.com/downloads_mscomp.html) 1.32：Doxygen 使用这个工具可以生成 CHM 格式的文档

## 使用

### 命令行
[下载链接；解压文件夹，添加环境变量](https://phoenixnap.dl.sourceforge.net/project/doxygen/rel-1.9.0/doxygen-1.9.0.windows.x64.bin.zip)

```shell
# 生成默认的配置文件
doxygen -g
# 开始生成
doxygen .\Doxyfile
```

### GUI

[下载链接；下载运行安装](https://doxygen.nl/files/doxygen-1.9.1-setup.exe)

  

1.  [C#使用](https://www.cnblogs.com/zhaoqingqing/p/3842236.html)
2.  [Doxygen + graphviz + Windows Help Workshop生成函数调用图和chm文件](https://blog.csdn.net/imgcl/article/details/79480350)