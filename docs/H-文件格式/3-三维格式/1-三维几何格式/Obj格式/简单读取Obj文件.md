```cpp
// - 打开文件
FILE * file = fopen(path, "r");
if( file == NULL )
{
    printf("Impossible to open the file !n");
    return false;
}

// - 读文件直到文件末尾
while( 1 )
{
    // - 读一行的第一个单词
    char lineHeader[128];	//假设第一行文字长度不超过128
    int res = fscanf(file, "%s", lineHeader);
    if (res == EOF)
        break; // EOF = End Of File. Quit the loop.
	else:
    	// 解析lineHeader
   
	// 处理顶点
    if ( strcmp( lineHeader, "v" ) == 0 )
    {
        // 若第一个字是'v'，则后面一定是3个float
        glm::vec3 vertex;
        fscanf(file, "%f %f %fn", &vertex.x, &vertex.y, &vertex.z );
        temp_vertices.push_back(vertex);
    }
    else if ( strcmp( lineHeader, "vt" ) == 0 )
    {
        // 若第一个字是'vt'，则后面一定是2个float
        glm::vec2 uv;
        fscanf(file, "%f %fn", &uv.x, &uv.y );
        temp_uvs.push_back(uv);
    }
    else if ( strcmp( lineHeader, "vn" ) == 0 )
    {
        //处理法线
        glm::vec3 normal;
        fscanf(file, "%f %f %fn", &normal.x, &normal.y, &normal.z );
        temp_normals.push_back(normal);
    }
    else if ( strcmp( lineHeader, "f" ) == 0 )
    {
        // 处理面
        std::string vertex1, vertex2, vertex3;
        unsigned int vertexIndex[3], uvIndex[3], normalIndex[3];
        int matches = fscanf(file, "%d/%d/%d %d/%d/%d %d/%d/%dn", 
	        &vertexIndex[0], &uvIndex[0], &normalIndex[0], 
	        &vertexIndex[1], &uvIndex[1], &normalIndex[1], 
	        &vertexIndex[2], &uvIndex[2], &normalIndex[2]
	    );
	    
        if (matches != 9){
            printf("File can't be read by our simple parser : ( Try exporting with other optionsn");
            return false;
        }
        
        vertexIndices.push_back(vertexIndex[0]);
        vertexIndices.push_back(vertexIndex[1]);
        vertexIndices.push_back(vertexIndex[2]);
        uvIndices    .push_back(uvIndex[0]);
        uvIndices    .push_back(uvIndex[1]);
        uvIndices    .push_back(uvIndex[2]);
        normalIndices.push_back(normalIndex[0]);
        normalIndices.push_back(normalIndex[1]);
        normalIndices.push_back(normalIndex[2]);
    }
}
```