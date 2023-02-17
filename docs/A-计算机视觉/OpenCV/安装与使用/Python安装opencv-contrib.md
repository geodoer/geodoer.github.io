Contrib版本的OpenCV是普通OpenCV的超集，包含了所有OpenCV正常版的功能。因此在安装前，需要卸载`opencv-python`
```shell
pip uninstall opencv-python
pip install opencv-contrib-python
```

## 示例
> 参考文章：[OpenCV Contrib扩展模块介绍与安装(Python) (zhaoxuhui.top)](http://zhaoxuhui.top/blog/2018/05/18/OpenCV_Contrib.html)

使用`cv2.xfeatures2d`模块中SIFT和SURF算子的简单示例代码

```python
# coding=utf-8
import cv2
import numpy as np

# 读取图像
img = cv2.imread("camera.png")

# 创建对象
# 对于SIFT算子，可以通过nFeatures属性控制特征点数量
# 对于SURF算子，可以通过hessianThreshold属性控制特征点数量
# 更详细的用法见OpenCV官方API文档
SIFT = cv2.xfeatures2d_SIFT.create()
SURF = cv2.xfeatures2d_SURF.create()

# 提取特征并计算描述子
kps, des = cv2.xfeatures2d_SIFT.detectAndCompute(SIFT, img, None)
kps2, des2 = cv2.xfeatures2d_SURF.detectAndCompute(SURF, img, None)

# 新建一个空图像用于绘制特征点
img_sift = np.zeros(img.shape, np.uint8)
img_surf = np.zeros(img.shape, np.uint8)

# 绘制特征点
cv2.drawKeypoints(img, kps, img_sift)
cv2.drawKeypoints(img, kps2, img_surf)

# 展示
cv2.imshow("img", img)
cv2.imshow("sift", img_sift)
cv2.imshow("surf", img_surf)
cv2.waitKey(0)
```