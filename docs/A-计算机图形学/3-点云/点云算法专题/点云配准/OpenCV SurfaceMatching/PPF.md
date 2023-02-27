---
title: OpenCV SurfaceMatching PPF
---



## PPF API介绍
> [PPF3DDetector API](https://docs.opencv.org/master/db/d25/classcv_1_1ppf__match__3d_1_1PPF3DDetector.html)


### 构造函数
```cpp
/**
 * @brief: 创建3D PPF 特征检测器，并指定相关参数

 * @param relativeSamplingStep: 相对于模型直径的采样步长
	在对模型进行模型描述建模时，用于在模型上采样的步长
		值越小，模型越稠密，姿态估计越准确，但是内存要求以及训练时间越长
		值越大，模型越稀疏，姿态估计精度降低，但是内存要求以及训练时间和匹配时间更短。
		
 * @param relativeDistanceStep: 相对于模型直径的离散步长
	在对模型进行模型描述建模时，用于对PPF特征向量进行离散化的步长
		值越小，则离散化越精细，哈希表越大，但是哈希表每个bin之间的关系越模糊
		值越大，细化越粗糙，哈希表越小，但是两个不同PPF特征向量可能会因为过大的步长而被放
 入相同的哈希槽中。
		默认该值与 'relativeSamplingStep'相同
		对于存在较多噪声的场景，该参数可以设得较大以提高对噪点的鲁棒性

  *@param numAngles:
	  在PPF特征检测的'voting scheme' 步骤中，需要对角度进行离散化，从而得以使用GHT算法。
	  角度离散化的区间数即为 'numAngles'。
	  参考论文中建议值为'30'，对于存在较多噪声的场景，可以将该参数设为 '25' 或 '20' 以提高对噪点的鲁棒性。
 */
cv::ppf_match_3d::PPF3DDetector::PPF3DDetector ( 
    const double 	relativeSamplingStep,
	const double 	relativeDistanceStep = 0.05,
	const double 	numAngles = 30 )
```

### 训练模型
```cpp
/**
 * @brief: 使用输入的 'Model' 数据，建立一个新的模型
 * @param Model: 模型的3D坐标+法向量点集(Nx6, CV_32F)
 */
void cv::ppf_match_3d::PPF3DDetector::trainModel (const Mat &Model)
```

### 匹配
```cpp
/**
 * @brief: 在提供的场景 'scene' 中，使用以训练的模型进行匹配，并返回匹配得到的所有可能姿态

 * @param scene: 目标场景的3D坐标+法向量点集(Nx6, CV_32F)

 * @param results: 最终求得的姿态列表。

 * @param relativeSceneSampleStep: 
	 相对于场景点集数量的采样步长。
	 如果设为 1.0/5.0，则场景点集中的 5-th 的点被用于计算。该参数提供了一种调整算法速度和精度的方法。较大的值可以提高速度，但是降低精度。反之，较小的值会提高精度，但降低速度。
 
 * @param relativeSceneDistance: 
	 相对于模型直径的距离阈值。参数作用类似于训练过程中的'relativeSamplingStep' 参数的作用。
 */
void cv::ppf_match_3d::PPF3DDetector::match	(	
    const Mat & 	scene,
	std::vector< Pose3DPtr > & 	results,
	const double 	relativeSceneSampleStep = 1.0/5.0,
	const double 	relativeSceneDistance = 0.03 )	
```