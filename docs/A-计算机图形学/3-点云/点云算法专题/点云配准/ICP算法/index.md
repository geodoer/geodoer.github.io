---
title: Iterative Closest Point  
tags:  
  - ICP  
  - 点云匹配  
---

## ICP简介
ICP常用的点云精配准算法。 

- ICP, Iterative Closest Point, 迭代最近点。顾名思义，ICP是通过最近邻法来找 **配对点** 的
- ICP经典论文：[A method for registration of 3-D shapes | IEEE Journals & Magazine | IEEE Xplore](https://ieeexplore.ieee.org/document/121791)

优缺点

- 需要提供一个较好的初值，同时由于算法本身缺陷，最终迭代结果可能会陷入局部最优

## 原理简介
ICP 算法流程

1. 首先，找到 Source 点云与 Target 点云中的对应点（corresponding point sets）
2. 然后，针对对应点，通过最小二乘法构建目标函数，进行迭代优化

### 估计对应点

对Source点云中的一点，求解其与Target点云中距离最近的那个点，作为其对应点。如此操作很费时间，因此通常会通过一些办法来加速

1. 阈值判断：可以设定一个阈值，当距离小于阈值时，就将其作为对应点
2. ANN(Approximate Nearest Neighbor)

### 迭代优化

1. 在“估计对应点”之后，就获得了两组对应的点集（N个点对）
2. 然后根据N个点对，计算出欧式变换$R$、$t$
3. 基于最小二乘法进行迭代计算，使得误差平方和达到极小值，从而得到最优的$R$、$t$

## 相关库

| 库名 | 说明 | 相关链接 |
| - | - | -|
| PCL | | |
| OpenCV SurfaceMatching ICP | |
| libpointmatcher | |[libpointmatcher](https://github.com/ethz-asl/libpointmatcher) |
| SLAM6D | | [SLAM6D 3DTK](https://slam6d.sourceforge.io/)|
| libicp | | [Andreas Geiger (cvlibs.net)](https://www.cvlibs.net/software/libicp/)|
| g-icp | | [avsegal/gicp: Generalized ICP reference implementation (github.com)](https://github.com/avsegal/gicp)|
| n-icp | | [Normal ICP - Normal Iterative Closest Point (jacoposerafin.com)](http://jacoposerafin.com/nicp/)|


## 参考链接

1. [【点云精配准】Iterative Closest Point（ICP） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/107218828)

