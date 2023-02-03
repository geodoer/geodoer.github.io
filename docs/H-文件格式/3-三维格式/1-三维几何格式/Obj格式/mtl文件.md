
## 简介
文件介绍

1. [mtl文件详解](http://paulbourke.net/dataformats/mtl/)

mtl，全称Material Library File，材质库文件。

1. mtl包含一个或多个材质的定义，每个定义均包含各个材质的颜色、纹理和反射贴图(reflection map)。这些将应用于对象的表面和顶点
2. 材质文件以ASCII格式存储，并具有.mtl扩展名
3. mtl文件与其他Alias|Wavefront属性文件（如light和atmosphere文件）不同在于：mtl可以包含多个材质定义，而其他属性文件只能包含一项定义

  

# 材质参数

## 基础材质
### 环境光
The [ambient](https://en.wikipedia.org/wiki/Shading#Ambient_lighting) color of the material is declared using Ka. Color definitions are in RGB where each channel's value is between 0 and 1.

```
# white
Ka 1.000 1.000 1.000
```

### 漫反射
Similarly, the [diffuse](https://en.wikipedia.org/wiki/Diffuse_reflection) color is declared using Kd.

```
# white
Kd 1.000 1.000 1.000
```

  

### 高光
> The [specular](https://en.wikipedia.org/wiki/Specular_reflection) color is declared using Ks, and weighted using the [specular exponent](https://en.wikipedia.org/wiki/Specular_exponent) Ns.

`Ks`指定高光（镜面反射）颜色。

`Ns`指定亮度，即`shininess`，一般[0, 1000]

```
# black (off)
Ks 0.000 0.000 0.000
# ranges between 0 and 1000
Ns 10.000
```

### 透明度
Materials can be [transparent](https://en.wikipedia.org/wiki/Transparency_and_translucency). This is referred to as being dissolved. Unlike real transparency, the result does not depend upon the thickness of the object. A value of 1.0 for "d" is the default and means fully opaque, as does a value of 0.0 for "Tr". Dissolve works on all illumination models.

```
# some implementations use 'd'
d 0.9
# others use 'Tr' (inverted: Tr = 1 - d)
Tr 0.1
```

> Transparent materials can additionally have a Transmission Filter Color, specified with "Tf".

透明材料还可以有透射滤镜颜色，由`Tf`指定。

```
# Transmission Filter Color (using R G B)
Tf 1.0 0.5 0.5

# Transmission Filter Color (using CIEXYZ) - y and z values are optional and assumed to be equal to x if omitted
Tf xyz 1.0 0.5 0.5

# Transmission Filter Color from spectral curve file (not commonly used)
Tf spectral <filename>.rfl <optional factor>
```

### 折射率
> A material can also have an optical density for its surface. This is also known as [index of refraction](https://en.wikipedia.org/wiki/Refractive_index).

材料的表面也可以有光密度，也被称之为折射率。
```
# optical density
Ni 1.45000
```

> Values can range from 0.001 to 10. A value of 1.0 means that light does not bend as it passes through an object. Increasing the optical density increases the amount of bending. Glass has an index of refraction of about 1.5. Values of less than 1.0 produce bizarre results and are not recommended.[[7]](https://en.wikipedia.org/wiki/Wavefront_.obj_file#cite_note-7)

取值范围为0.001 ~ 10。值为1.0意味着光在穿过物体时不会弯曲。增加光密度会增加弯曲量。玻璃的折射率约为1.5。小于1.0的值会产生奇怪的结果，不建议使用

### 照明模型
```
illum 2
```

> Multiple [illumination models](https://en.wikipedia.org/wiki/Illumination_model) are available, per material. Note that it is not required to set a transparent illumination model in order to achieve transparency with "d" or "Tr", and in modern usage illum models are often not specified, even with transparent materials. The illum models are enumerated as follows:

```
0. Color on and Ambient off
1. Color on and Ambient on
2. Highlight on
3. Reflection on and Ray trace on
4. Transparency: Glass on, Reflection: Ray trace on
5. Reflection: Fresnel on and Ray trace on
6. Transparency: Refraction on, Reflection: Fresnel off and Ray trace on
7. Transparency: Refraction on, Reflection: Fresnel on and Ray trace on
8. Reflection on and Ray trace off
9. Transparency: Glass on, Reflection: Ray trace off
10. Casts shadows onto invisible surfaces
```

## 纹理映射
在基础材质中，是用一个参数值来描述，但也可以用一个纹理来做映射。

> Textured materials use the same properties as above, and additionally define [texture maps](https://en.wikipedia.org/wiki/Texture_mapping). Below is an example of a common material file. See the full Wavefront file format reference for more details.

例如：
```
newmtl Textured
   Ka 1.000 1.000 1.000
   Kd 1.000 1.000 1.000
   Ks 0.000 0.000 0.000
   d 1.0
   illum 2
   
   # the ambient texture map
   map_Ka lemur.tga

   # the diffuse texture map (most of the time, it will be the same as the
   # ambient texture map)
   map_Kd lemur.tga

   # specular color texture map
   map_Ks lemur.tga

   # specular highlight component
   map_Ns lemur_spec.tga

   # the alpha texture map
   map_d lemur_alpha.tga

   # some implementations use 'map_bump' instead of 'bump' below
   map_bump lemur_bump.tga

   # bump map (which by default uses luminance channel of the image)
   bump lemur_bump.tga

   # displacement map
   disp lemur_disp.tga

   # stencil decal texture (defaults to 'matte' channel of the image)
   decal lemur_stencil.tga
```

> Texture map statements may also have option parameters (see [full spec](http://paulbourke.net/dataformats/mtl/)).

```
   # texture origin (1,1,1)
   map_Ka -o 1 1 1 ambient.tga

   # spherical reflection map
   refl -type sphere clouds.tga
```

## 纹理选项
```
-blendu on | off                       # set horizontal texture blending (default on)
-blendv on | off                       # set vertical texture blending (default on)
-boost float_value                     # boost mip-map sharpness
-mm base_value gain_value              # modify texture map values (default 0 1)
                                       #     base_value = brightness, gain_value = contrast
-o u [v [w]]                           # Origin offset             (default 0 0 0)
-s u [v [w]]                           # Scale                     (default 1 1 1)
-t u [v [w]]                           # Turbulence                (default 0 0 0)
-texres resolution                     # texture resolution to create
-clamp on | off                        # only render texels in the clamped 0-1 range (default off)
                                       #   When unclamped, textures are repeated across a surface,
                                       #   when clamped, only texels which fall within the 0-1
                                       #   range are rendered.
-bm mult_value                         # bump multiplier (for bump maps only)
-imfchan r | g | b | m | l | z         # specifies which channel of the file is used to
                                       # create a scalar or bump texture. r:red, g:green,
                                       # b:blue, m:matte, l:luminance, z:z-depth..
                                       # (the default for bump is 'l' and for decal is 'm')
```

例子
```
# says to use the red channel of bumpmap.tga as the bumpmap
bump -imfchan r bumpmap.tga
```

> For [reflection maps](https://en.wikipedia.org/wiki/Reflection_mapping)...
```
-type sphere                           # specifies a sphere for a "refl" reflection map
-type cube_top    | cube_bottom |      # when using a cube map, the texture file for each
      cube_front  | cube_back   |      # side of the cube is specified separately
      cube_left   | cube_right
```

## 其他选项
### pbr
# 数据结构
来源于tiny_obj_loader的源码

```cpp
struct material_t {
    std::string name;
    real_t ambient[3];  //Ka 环境光 [0,1]
    real_t diffuse[3];  //Kd 漫反射 [0,1]
    real_t specular[3]; //Ks 镜面反射（高光） [0,1]
    real_t transmittance[3];  //
    real_t emission[3];
    real_t shininess;
    real_t ior;       // index of refraction
    real_t dissolve;  // 1 == opaque; 0 == fully transparent
    // illumination model (see http://www.fileformat.info/format/material/)
    int illum;
    int dummy;  // Suppress padding warning.
    std::string ambient_texname;             // map_Ka
    std::string diffuse_texname;             // map_Kd
    std::string specular_texname;            // map_Ks
    std::string specular_highlight_texname;  // map_Ns
    std::string bump_texname;                // map_bump, map_Bump, bump
    std::string displacement_texname;        // disp
    std::string alpha_texname;               // map_d
    std::string reflection_texname;          // refl
    texture_option_t ambient_texopt;
    texture_option_t diffuse_texopt;
    texture_option_t specular_texopt;
    texture_option_t specular_highlight_texopt;
    texture_option_t bump_texopt;
    texture_option_t displacement_texopt;
    texture_option_t alpha_texopt;
    texture_option_t reflection_texopt;

    // PBR extension
    // http://exocortex.com/blog/extending_wavefront_mtl_to_support_pbr
    real_t roughness;            // [0, 1] default 0
    real_t metallic;             // [0, 1] default 0
    real_t sheen;                // [0, 1] default 0
    real_t clearcoat_thickness;  // [0, 1] default 0
    real_t clearcoat_roughness;  // [0, 1] default 0
    real_t anisotropy;           // aniso. [0, 1] default 0
    real_t anisotropy_rotation;  // anisor. [0, 1] default 0
    real_t pad0;

    std::string roughness_texname;  // map_Pr
    std::string metallic_texname;   // map_Pm
    std::string sheen_texname;      // map_Ps
    std::string emissive_texname;   // map_Ke
    std::string normal_texname;     // norm. For normal mapping.

    texture_option_t roughness_texopt;
    texture_option_t metallic_texopt;
    texture_option_t sheen_texopt;
    texture_option_t emissive_texopt;
    texture_option_t normal_texopt;
    int pad2;
    std::map<std::string, std::string> unknown_parameter;
};
```

# 文件组织
.mtl文件的组织如下：

```
   newmtl my_red
      Material color
      & illumination
      statements
      texture map
      statements
      reflection map
      statement
   newmtl my_blue
      Material color
      & illumination
      statements
      texture map
      statements
      reflection map
      statement
   newmtl my_green
      Material color
      & illumination
      statements
      texture map
      statements
      reflection map
      statement
```

obj挂接纹理
```
#引入材质文件
mtllib 1.mtl

#在Face前引用
usemtl mat1
f 1 2 3
```

# 示例
#### 霓虹绿
这是一种亮绿色的材料。应用于对象时，无论场景中的任何灯光，它都将保持亮绿色。

```
newmtl neon_green
Kd 0.0000 1.0000 0.0000
illum 0
```

  

#### 平坦的绿色
这是平坦的绿色材料。

```
newmtl flat_green
Ka 0.0000 1.0000 0.0000
Kd 0.0000 1.0000 0.0000
illum 1
```

#### 溶解的绿色

这是一种纯绿色的，部分溶解的材料。

```
newmtl diss_green
Ka 0.0000 1.0000 0.0000
Kd 0.0000 1.0000 0.0000
d 0.8000
illum 1
```

#### 闪亮的绿色

这是闪亮的绿色材料。应用于对象时，它将显示白色镜面高光。

```
newmtl Shiny_green
Ka 0.0000 1.0000 0.0000
Kd 0.0000 1.0000 0.0000
Ks 1.0000 1.0000 1.0000
Ns 200.0000
illum 1
```

#### 绿镜

这是一种反射性的绿色材料。当应用于对象时，它将反映同一场景中的其他对象。

```
newmtl green_mirror
Ka 0.0000 1.0000 0.0000
Kd 0.0000 1.0000 0.0000
Ks 0.0000 1.0000 0.0000
Ns 200.0000
illum 3
```

#### 假挡风玻璃

此材料近似于玻璃表面。它几乎是完全透明的，但是可以显示场景中其他对象的反射。它不会使透过材料看到的物体的图像变形。

```
newmtl fake_windsh
Ka 0.0000 0.0000 0.0000
Kd 0.0000 0.0000 0.0000
Ks 0.9000 0.9000 0.9000
d 0.1000
Ns 200
illum 4
```

#### 菲涅耳蓝
这种材料表现出一种称为菲涅耳反射的效果。当施加到物体，白色条纹可能会出现在对象的以掠射角度查看表面。

```
newmtl fresnel_blu
Ka 0.0000 0.0000 0.0000
Kd 0.0000 0.0000 0.0000
Ks 0.6180 0.8760 0.1430
Ns 200
illum 5
```

#### 真实的挡风玻璃

该材料精确地代表了玻璃表面。它过滤通过它看到的对象的颜色。根据材料的透射色进行过滤。该材料还会根据其光密度扭曲对象的图像。请注意，该材料未溶解，并且其环境色，漫反射色和镜面反射色已设置为黑色。只有透射色是非黑色的。

```
newmtl real_windsh
Ka 0.0000 0.0000 0.0000
Kd 0.0000 0.0000 0.0000
Ks 0.0000 0.0000 0.0000
Tf 1.0000 1.0000 1.0000
Ns 200
Ni 1.2000
illum 6
```

#### 菲涅耳挡风玻璃
该材料结合了示例7和8中的效果

```
newmtl fresnel_win
Ka 0.0000 0.0000 1.000
Kd 0.0000 0.0000 1.0000
Ks 0.6180 0.8760 0.1430
Tf 1.0000 1.0000 1.0000
Ns 200
Ni 1.2000
illum 7
```
