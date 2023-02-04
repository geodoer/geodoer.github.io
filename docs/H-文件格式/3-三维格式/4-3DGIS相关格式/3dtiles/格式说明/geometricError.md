geometricError就是一个对象外包球的直径，会根据相机位置计算投影屏幕所站的像素大小

几何度量误差，Geometric Error，简称 GE，是计算机图形图像学领域中用来描述计算机绘制的近似几何模型与理想之间近似程度的一种度量误差。

## 示例
### get_geometric_error
[https://github.com/fanvanzh/3dtiles](https://github.com/fanvanzh/3dtiles) osgb23dtile.cpp

```cpp
struct TileBox
{
    std::vector<double> max;
    std::vector<double> min;
    
    void extend(double ratio) {
        ratio /= 2;
        double x = max[0] - min[0];
        double y = max[1] - min[1];
        double z = max[2] - min[2];
        max[0] += x * ratio;
        max[1] += y * ratio;
        max[2] += z * ratio;
    
        min[0] -= x * ratio;
        min[1] -= y * ratio;
        min[2] -= z * ratio;
    }
};

double get_geometric_error(TileBox& bbox)
{
    if (bbox.max.empty() || bbox.min.empty())
    {
        LOG_E("bbox is empty!");
        return 0;
    }

    double max_err = std::max((bbox.max[0] - bbox.min[0]),(bbox.max[1] - bbox.min[1]));
    max_err = std::max(max_err, (bbox.max[2] - bbox.min[2]));
    return max_err / 20.0;
    //     const double pi = std::acos(-1);
    //     double round = 2 * pi * 6378137.0 / 128.0;
    //     return round / std::pow(2.0, lvl );
}
```

## 参考链接

1.  [Cesium3DTile中的getScreenSpaceError方法及公式分析](https://blog.csdn.net/Rsoftwaretest/article/details/106740269)
2.  [Geometric Error与Screen Space Error](https://blog.csdn.net/whl0071/article/details/126041237)