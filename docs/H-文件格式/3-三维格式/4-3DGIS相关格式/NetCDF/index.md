---
title: NetCDF
---

NetCDF (Network Common Data Form，网络通用数据格式)是一组软件库和独立于机器的数据格式

- 支持面向数组的科学数据的创建、访问与共享
- 它也是共享科学数据的社区标准
- netcdf文件开始的目的是用于存储气象科学中的数据，现在已经成为许多数据采集软件的生成文件的格式
- 项目主页由[University Corporation for Atmospheric Research](https://en.wikipedia.org/wiki/University_Corporation_for_Atmospheric_Research) (UCAR)的Unidata项目托管

相关网站

1. [NetCDF用户指南](https://docs.unidata.ucar.edu/nug/current/)
2. [Network Common Data Form (NetCDF)](https://www.unidata.ucar.edu/software/netcdf/)

## 文件内容
从数学上来说，netcdf存储的数据就是一个多自变量的单值函数。

- 用公式来说就是f(x,y,z,…)=value, 函数的自变量x,y,z等在netcdf中叫做维(dimension)或坐标轴(axix)
- 函数值value在netcdf中叫做变量(Variables)
- 而自变量和函数值在物理学上的一些性质，比如计量单位(量纲)、物理学名称等等，在netcdf中就叫属性(Attributes)

### 变量
netcdf中的变量就是一个N维数组，数组的维数就是实际问题中的自变量个数，数组的值就是观测得到的物理值。变量（数组值）在netcdf中的存储类型有六种，ascii字符(char) ,字节(byte), 短整型(short), 整型(int), 浮点(float), 双精度(double)。

### 维
一个维对应着函数中的某个自变量，或者说函数图象中的一个坐标轴，在线性代数中就是一个N维向量的一个分量（这也是维这个名称的由来）。

在netcdf中，一个维具有一个名字和范围（或者说长度，也就是数学上所说的定义域，可以是离散的点集合或者连续的区间）。在netcdf中,维的长度基本都是有限的，最多只能有一个具有无限长度的维。

### 属性
属性对变量值和维的具体物理含义的注释或者说解释。因为变量和维在netcdf中都只是无量纲的数字，要想让人们明白这些数字的具体含义，就得靠属性这个对象了。
在netcdf中，属性由一个属性名和一个属性值（一般为字符串）组成。

比如，在某个cdl文件(cdl文件的具体格式在下一节中讲述)中有这样的代码段

- 前面的temperature是一个已经定义好的变量（Variable），即温度
- 冒号后面的units就是属性名，表示物理单位，=后面的就是units这个属性的值，为“celsius” ，即摄氏度
- 整个一行代码的意思就是温度这个物理量的单位为celsius，很好理解。
```
temperature:units = "celsius";
```


## CDL结构
CDL全称为network Common data form Description Language，它是用来描述netcdf文件的结构的一种语法格式。

示例：netcdf教程 2.1  The simple xy Example

1. 维，以`dimensions:`开头，定义了xy两个轴，并标明了个数
2. 变量，以`variables:`开头，定义了一个以x轴和y轴为自变量的函数data，数学公式就是f(x,y)=data
3. 数据的定义，以`data:`开头，`data=...`定义了一个二维的数组（一共有两行）
   1. x=0, y=0，data=f(x,y)=0
   2. x=0, y=1, data=f(x,y)=1
   3. x=5,y=11, data=f(x,y)=71
```
netcdf simple_xy {
dimensions:
x = 6 ;
y = 12 ;
variables:
int data(x, y) ;
data:
data =
0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11,
12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23,
24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35,
36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47,
48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59,
60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71 ;
}
```

## 格式说明
NetCDF 库支持 NetCDF 文件的多种不同二进制格式：

- Classic格式在第一个 NetCDF 版本中使用，并且仍然是文件创建的默认格式。
- 64 位偏移格式是在 3.6.0 版本中引入的，它支持更大的变量和文件大小。
- NetCDF-4/HDF5 格式是在 4.0 版本中引入的；它是 HDF5 数据格式，有一些限制。
- HDF4 SD 格式支持只读访问。
- 支持 CDF5 格式，与 parallel-netcdf 项目协调。

## Unidata
Unidata程序中心支持和维护C、C++、Java和Forton的NetCDF编程接口。它的编程接口也可用于Python、IDL、Matlab、R、Ruby和Perl等语言。
可参考链接：[unidata netcdf](https://www.unidata.ucar.edu/software/netcdf/)
仓库地址：[https://github.com/Unidata/netcdf-cxx4](https://github.com/Unidata/netcdf-cxx4)

编程接口的优点：

- Self-Describing：包含元数据，以对自己进行描述
- Portable：计算机可以通过不同的方式存储整数、字符和浮点数来访问NetCDF文件
- Scalable：可以通过NetCDF接口有效地访问各种格式的大型数据集的小子集，甚至可以从远程服务器访问
- Appendable：可以将数据附加到结构中，而无需复制数据集或重新定义其结构
- Sharable：一个写入器和多个读取器可同时访问一个NetCDF文件
- Archivable：支持早期NetCDF格式

## 参考文章

1. [Network Common Data Form (NetCDF)](https://www.unidata.ucar.edu/software/netcdf/)
2. [wiki NetCDF](https://en.wikipedia.org/wiki/NetCDF)
3. [NC文件读写方式](https://blog.csdn.net/ennaymonkey/article/details/62886843)
