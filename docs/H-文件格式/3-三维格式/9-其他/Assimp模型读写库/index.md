---
title: Assimp模型读写库
---

相关链接

1. [官方文档](http://assimp.sourceforge.net/lib_html/)
2. [用Assimp加载模型](https://www.cnblogs.com/zhlabcd/p/11727712.html)
3. [使用案例-导入导出](https://blog.csdn.net/Johnable/article/details/101422870)

## 简介
Assimp是**Open Asset Import Library**（开放的资产导入库-）的缩写，能够导入很多种不同的模型文件格式（也支持导出部分格式），它会将所有的模型数据加载至Assimp的通用数据结构中。

### 支持导入的格式

- Autodesk ( .fbx )
- Collada ( .dae )
- glTF ( .gltf, .glb )
- Blender 3D ( .blend )
- 3ds Max 3DS ( .3ds )
- 3ds Max ASE ( .ase )
- Wavefront Object ( .obj )
- Industry Foundation Classes (IFC/Step) ( .ifc )
- XGL ( .xgl,.zgl )
- Stanford Polygon Library ( .ply )
- *AutoCAD DXF ( .dxf )
- LightWave ( .lwo )
- LightWave Scene ( .lws )
- Modo ( .lxo )
- Stereolithography ( .stl )
- DirectX X ( .x )
- AC3D ( .ac )
- Milkshape 3D ( .ms3d )
- * TrueSpace ( .cob,.scn )
- Biovision BVH ( .bvh )
- * CharacterStudio Motion ( .csm )
- Ogre XML ( .xml )
- Irrlicht Mesh ( .irrmesh )
- * Irrlicht Scene ( .irr )
- Quake I ( .mdl )
- Quake II ( .md2 )
- Quake III Mesh ( .md3 )
- Quake III Map/BSP ( .pk3 )
- * Return to Castle Wolfenstein ( .mdc )
- Doom 3 ( .md5* )
- *Valve Model ( .smd,.vta )
- *Open Game Engine Exchange ( .ogex )
- *Unreal ( .3d )
- BlitzBasic 3D ( .b3d )
- Quick3D ( .q3d,.q3s )
- Neutral File Format ( .nff )
- Sense8 WorldToolKit ( .nff )
- Object File Format ( .off )
- PovRAY Raw ( .raw )
- Terragen Terrain ( .ter )
- 3D GameStudio (3DGS) ( .mdl )
- 3D GameStudio (3DGS) Terrain ( .hmp )
- Izware Nendo ( .ndo )

### 支持导出的格式
```cpp
Assimp::Exporter exporter;
for (int i = 0; i < exporter.GetExportFormatCount(); ++i) {
    cout << "第" << i << "种格式：" << endl;
    const aiExportFormatDesc* fmt_desc = exporter.GetExportFormatDescription(i);
    cout << fmt_desc->id << endl;
    cout << fmt_desc->fileExtension << endl;
    cout << fmt_desc->description << endl;
    cout << endl;
}
```

- Collada ( .dae )
- Wavefront Object ( .obj )
- Stereolithography ( .stl )
- Stanford Polygon Library ( .ply )

