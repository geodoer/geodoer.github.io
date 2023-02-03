---
title: Ply格式
---

「ply」Polygon File Format（多边形文件格式）或Stanford Triangle Format（斯坦福三角形档案）

## 示例
### 模型

```text
ply                 
format ascii 1.0   
	//数据保存格式，三类
	//format ascii 1.0
	//format binary_little_endian 1.0
	//format binary_big_endian 1.0
element vertex 8 //元素：顶点 个数为8
property float x    
property float y    
property float z    //顶点格式：依次为x,yz坐标
element edge 6  //元素：边 6条
property int vertex1
property int vertex2
property uchar red  
property uchar green
property uchar blue //边存储格式为： 顶点id 1，2，颜色r,g,b
end_header  //头，以end_header结束
0.194585 0.0202505 -0.654565
0.393574 0.0181872 -0.634588
0.196413 0.220227 -0.652125
0.174584 0.0180056 -0.455581
0.811062 -0.0294865 -0.551833
0.991697 -0.0650619 -0.473697
0.845413 0.167279 -0.541659
0.73238 -0.0252545 -0.368009  //点内容，8个顶点（x,y,z）坐标
0 1 255 0 0
0 2 0 255 0
0 3 0 0 255
4 5 255 0 0
4 6 0 255 0
4 7 0 0 255   //6条边，（id1,id2,r,g,b）
```

### 点云
```
ply
format ascii 1.0
element vertex %(vert_num)
property float x
property float y
property float z
property uchar red
property uchar green
property uchar blue
end_header
```