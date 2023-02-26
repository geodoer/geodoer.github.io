---
title: Triangle(2D三角化)
---

相关链接
1. [https://www.cs.cmu.edu/~quake/triangle.html](https://www.cs.cmu.edu/~quake/triangle.html)；[代码](https://github.com/geodoer/triangle)
2. Triangle的封装：[wo80/Triangle](https://github.com/wo80/Triangle)

## 命令行参数
文档：[Triangle: Command line switches (cmu.edu)](https://www.cs.cmu.edu/~quake/triangle.switch.html)

```
-p Triangulates a Planar Straight Line Graph (.poly file).
	输入一个.poly（平面直线图）文件，对它进行三角化
	
-r Refines a previously generated mesh.
	优先之前生成的网格

-q Quality mesh generation with no angles smaller than 20 degrees. An alternate minimum angle may be specified after the `q'.
	生成不小于20°角度的高质量网格。可以在"q"之后指定最小角度

-a Imposes a maximum triangle area constraint. A fixed area constraint (that applies to every triangle) may be specified after the `a', or varying area constraints may be read from a .poly file or .area file.
	对三角形最大面积做约束

-u Imposes a user-defined constraint on triangle size.
	对三角形个数做约束

-A Assigns a regional attribute to each triangle that identifies what segment-bounded region it belongs to.
	为每个三角形分配一个区域属性

-c Encloses the convex hull with segments.
	用线段集包围凹包

-D Conforming Delaunay: use this switch if you want all triangles in the mesh to be Delaunay, and not just constrained Delaunay; or if you want to ensure that all Voronoi vertices lie within the triangulation.
	符合德劳内

-j Jettisons vertices that are not part of the final triangulation from the output .node file (including duplicate input vertices and vertices ``eaten'' by holes).
	放弃不属于结果中的顶点（如重复输入的顶点、在洞内的顶点）

-e Outputs (to an .edge file) a list of edges of the triangulation.
	将三角网的边输出到.edge文件中

-v Outputs the Voronoi diagram associated with the triangulation. Does not attempt to detect degeneracies, so some Voronoi vertices may be duplicated.
	输出与三角化相关的Voronoi图

-n Outputs (to a .neigh file) a list of triangles neighboring each triangle.
	将与每个三角形相邻的三角形输出到.neigh文件当中

-g Outputs the mesh to an Object File Format (.off) file, suitable for viewing with the Geometry Center's Geomview package.
	将三角网输出到off文件当中

-B Suppresses boundary markers in the output .node, .poly, and .edge output files.
	禁止将 边界标记 输出到node、poly、edge文件当中

-P Suppresses the output .poly file. Saves disk space, but you lose the ability to maintain constraining segments on later refinements of the mesh.
	禁止输出poly文件

-N Suppresses the output .node file.
	禁止输出node文件

-E Suppresses the output .ele file.
	禁止输出ele文件

-I Suppresses mesh iteration numbers.
	禁止网格迭代

-O Suppresses holes: ignores the holes in the .poly file.
	禁止将孔洞输出到Poly文件当中

-X Suppresses exact arithmetic.
	禁止精确算术

-z Numbers all items starting from zero (rather than one). Note that this switch is normally overrided by the value used to number the first vertex of the input .node or .poly file. However, this switch is useful when calling Triangle from another program.
	对所有从0开始（而不是从1开始）的项进行编码。

-o2 Generates second-order subparametric elements with six nodes each.
	生成二阶子参数，每个元素有六个节点

-Y Prohibits the insertion of Steiner points on the mesh boundary. If specified twice (-YY), it prohibits the insertion of Steiner points on any segment, including internal segments.
	禁止在网格边界上插入Steiner点。若指定两个Y（-YY），则禁止插入Steiner点在任何线段上，包括内部的线段。

-S Specifies the maximum number of added Steiner points.
	指定添加Steiner点的最大数量

-i Uses the incremental algorithm for Delaunay triangulation, rather than the divide-and-conquer algorithm.
	使用增量算法进行Delaunay三角化，而不是分而治之的算法

-F Uses Steven Fortune's sweepline algorithm for Delaunay triangulation, rather than the divide-and-conquer algorithm.
	使用Steven Fortune的扫描算法进行Delaunay三角化，而不是分而治之的算法

-l Uses only vertical cuts in the divide-and-conquer algorithm. By default, Triangle uses alternating vertical and horizontal cuts, which usually improve the speed except with vertex sets that are small or short and wide. This switch is primarily of theoretical interest.
	在分而治之算法中，仅使用垂直切割。默认情况下，三角形交替使用垂直、水平切割，这通常会提高速度，除了顶点集小、短和宽的情况

-s Specifies that segments should be forced into the triangulation by recursively splitting them at their midpoints, rather than by generating a constrained Delaunay triangulation. Segment splitting is true to Ruppert's original algorithm, but can create needlessly small triangles. This switch is primarily of theoretical interest.
	使用 递归拆分它们中点 的方法进行强制三角化，而不是生成受约束的德劳内三角化

-C Check the consistency of the final mesh. Uses exact arithmetic for checking, even if the -X switch is used. Useful if you suspect Triangle is buggy.
	检查最终网格的一致性

-Q Quiet: Suppresses all explanation of what Triangle is doing, unless an error occurs.
	禁止输出三角形正在做什么的信息，除非遇到错误

-V Verbose: Gives detailed information about what Triangle is doing. Add more `V's for increasing amount of detail. `-V' gives information on algorithmic progress and detailed statistics.
	详细输出，包括每个三角形在做什么的输出

-h Help: Displays complete instructions.
```


