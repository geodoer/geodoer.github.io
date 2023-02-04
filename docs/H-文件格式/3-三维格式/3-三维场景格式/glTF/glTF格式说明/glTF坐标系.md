
gltf的模型坐标系是Y轴向上的，因此它的坐标、矩阵都是基于此坐标系下的。  
在根节点上加上一个偏移矩阵，将模型坐标系对标到世界坐标系上（Z轴朝上）

```cpp
//# Z轴朝上
if(_do_zaxis_up)
{
	auto transform_node = new MatrixTransformNode();
	
	Matrixd gltf_to_world_matrix;
	gltf_to_world_matrix *= Matrixd::rotate(Math::radians(90.0f), Vec3f{ 1,0,0 });	//绕X旋转90°
	transform_node->setMatrix(gltf_to_world_matrix);

	transform_node->addChild(_root);
	_root = transform_node;
}
```

验证：与blender加载glTF的结果一致。

## 附：查看glTF坐标系的方法
查看方法：VSCode Gltf Tool工具 > 打开gltf文件 > alt+G打开预览 > 调整Position的XYZ即可发现XYZ

  根节点的局部坐标系
![](images/glTF根节点的局部坐标系.png)

每个根节点的偏移矩阵（根节点 -> 场景的偏移矩阵）

- 绕Y轴旋转180°
- Z sclae -1

![](images/glTF根节点的偏移矩阵1.png)

## 参考文章
1. [Cesium中gltf模型的坐标系](https://blog.csdn.net/u011575168/article/details/124016666)