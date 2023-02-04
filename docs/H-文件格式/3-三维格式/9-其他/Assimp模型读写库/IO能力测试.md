> 虽然说，Assimp能支持这么多格式，但支持的粒度、完整度，还是要测试的。


构建工程：直接用CMake，构建VS的工程即可。

### 测试方法
assimp_cmd子项目提供了基础的读写示例，通过命令行调用即可调试

测试例子：

1. 将assimp_cmd设置为启动项
2. 右键assimp_cmd > 属性 > 调试 > 命令参数，输入`info D:\test\assimp-5.0.1\data\assimp-mdb\model-db\Obj\Wuson\WusonOBJ.obj`
3. 运行即可调试

测试数据：在以下文件目录中有很多模型文件

1. assimp-5.0.1\data\assimp-mdb\model-db
2. assimp-5.0.1\test\models

参数说明：
```cpp
const char* AICMD_MSG_HELP = 
    "assimp <verb> <parameters>\n\n"
    " verbs:\n"
    " \tinfo       - Quick file stats\n"
    " \tlistext    - List all known file extensions available for import\n"
    " \tknowext    - Check whether a file extension is recognized by Assimp\n"
#ifndef ASSIMP_BUILD_NO_EXPORT
    " \texport     - Export a file to one of the supported output formats\n"
    " \tlistexport - List all supported export formats\n"
    " \texportinfo - Show basic information on a specific export format\n"
#endif
    " \textract    - Extract embedded texture images\n"
    " \tdump       - Convert models to a binary or textual dump (ASSBIN/ASSXML)\n"
    " \tcmpdump    - Compare dumps created using \'assimp dump <file> -s ...\'\n"
    " \tversion    - Display Assimp version\n"
    "\n Use \'assimp <verb> --help\' for detailed help on a command.\n"
;
```

### 导入OBJ，输出fbx
```
export D:\test\assimp-5.0.1\data\assimp-mdb\model-db\Obj\Wuson\WusonOBJ.obj  --format=FBX
```

### IFC测试
导入IFC，输出fbx
`export D:\test\assimp-5.0.1\data\assimp-mdb\model-db\ifc\1.ifc --format=FBX`

报错信息
```
ERROR: Failed to load file: (entity #25) type error reading literal field - expected argument 8 to IfcSpatialStructureElement to be a `IfcElementCompositionEnum`
```

- 参考文章：https://www.steptools.com/stds/ifc/html/t_ifcelementcompositionenum.html

测试数据中1.ifc，9.ifc比较复杂，都是上面的报错。
对于简单的wall.ifc，没有 `IfcSpatialStructureElement`或 `IfcElementCompositionEnum`构件类的，就可以读取成功

### gltf测试
初步测试读写正常

