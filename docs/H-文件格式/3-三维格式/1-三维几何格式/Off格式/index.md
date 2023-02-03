---
title: Off格式
---

物体文件格式（.off）文件通过描述物体表面的多边形来表示一个模型的几何结构，这里的多边形可以有任意数量的顶点。

普林斯顿形状 Banchmark（Princeton Shape Benchmark）中的 .off 文件遵循以下标准：

-   OFF文件全是以OFF关键字开始的ASCII文件。
-   下一行说明顶点的数量、面片的数量、边的数量。边的数量可以安全地省略。
-   顶点按每行一个列出x、y、z坐标。
-   在顶点列表后，面片按照每行一个列表，对于每个面片，顶点的数量是指定的，接下来是顶点索引列表。

## 示例

### 点云
```
OFF
顶点数 面片数 边数
x y z
x y z
...
n个顶点 顶点1的索引 顶点2的索引 … 顶点n的索引
...
```

### 模型
```
OFF
8 12 0
-0.274878 -0.274878 -0.274878
-0.274878  0.274878 -0.274878
 0.274878  0.274878 -0.274878
 0.274878 -0.274878 -0.274878
-0.274878 -0.274878  0.274878
-0.274878  0.274878  0.274878
 0.274878  0.274878  0.274878
 0.274878 -0.274878  0.274878
3  0 1 3
3  3 1 2
3  0 4 1
3  1 4 5
3  3 2 7
3  7 2 6
3  4 0 3
3  7 4 3
3  6 4 7
3  6 5 4
3  1 5 6
3  2 1 6
```

## 相关链接

1. [https://people.sc.fsu.edu/~jburkardt/data/off/off.html](https://people.sc.fsu.edu/~jburkardt/data/off/off.html)
2. [http://local.wasp.uwa.edu.au/~pbourke/dataformats/off/](http://local.wasp.uwa.edu.au/~pbourke/dataformats/off/)
3. [http://www.holmes3d.net/graphics/offfiles/offdesc.txt](http://www.holmes3d.net/graphics/offfiles/offdesc.txt)
4. [http://www.holmes3d.net/graphics/offfiles/](http://www.holmes3d.net/graphics/offfiles/)
5. [http://shape.cs.princeton.edu/benchmark/documentation/off_format.html](http://shape.cs.princeton.edu/benchmark/documentation/off_format.html)