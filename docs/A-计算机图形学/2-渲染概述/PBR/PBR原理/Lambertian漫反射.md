【Lambertian漫反射】

$$
f_{lambert} = \frac{c}{\pi}
$$

1. $c$表示表面颜色（回想一下漫反射表面纹理）
2. 除以$\pi$是为了对漫反射光进行标准化，因为前面含有BRDF的积分方程是受$\pi$影响的

【Lambertian的效率】

- 除此之外，其他BRDF有其他的漫反射方式，大多数看上去都相当真实，但是相应的运算开销也非常的昂贵。
- 不过按Epic公司给出的结论，Lambertian漫反射模型已经足够应付大多数实时渲染的用途了

## 参考链接

1. [PBR理论](https://learnopengl-cn.github.io/07%20PBR/01%20Theory/)
