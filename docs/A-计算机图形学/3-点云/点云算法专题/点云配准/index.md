---
title: 点云配准
---

## 点云配准分类
通常，点云配准分为两步，先做粗配准，再做精配准

-   粗配准（Coarse registration）：基于局部几何特征
-   精配准（Fine registration）：需要初始位姿（initial alignment）

| 方法 | 原理简介 | 优点 | 缺点 |
| - | - | -| - |
| Iterative Closest Point(ICP) | 迭代最近点依次匹配点对 | | 1.要剔除不合适的点对（距离过大、在边界上的点对）<br/>2. 基于点对的配准，不包含局部形状信息<br/>3. 每次迭代都要搜索最近点，耗时计算量大 |
| IDC | ICP的一种改进，采用极坐标代替笛卡尔坐标进行最近点搜索 |
| PIC | | 考虑了点云的噪音和初始位置的不确定性|
| Point-based probabilistic registration | 需要首先建立深度图的三角面片 |
| NDT<br/>正态分布变换 |  | 不需要反复去找最近点，只需要一次计算。概率密度函数在两幅图像采集之间的时间可以离线计算出来 |
| Gaussian fields | 和NDT类似，利用高斯混合模型考察点与点的距离和点周围表面的相似性 |
| Quadratic patches | | 
| Likelihood-field matching<br/>随机场匹配 | |
| CRF匹配 | | | 运行速度慢，在3d中实时性能不好，误差大 |
| Branch-and-bound registration | |
| Registration using local geometric features | |

## 参考文章
1. [点云配准综述 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/91275450)
2. [NDT 算法（与ICP对比）和一些常见配准算法](http://ghx0x0.github.io/2014/12/30/NDT-match/)

