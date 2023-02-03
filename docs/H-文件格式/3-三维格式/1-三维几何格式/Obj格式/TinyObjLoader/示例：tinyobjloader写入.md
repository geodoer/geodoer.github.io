
```cpp
std::cout << "# of vertices  : " << (attrib.vertices.size() / 3) << std::endl;
std::cout << "# of normals   : " << (attrib.normals.size() / 3) << std::endl;
std::cout << "# of texcoords : " << (attrib.texcoords.size() / 2) << std::endl;
std::cout << "# of shapes    : " << shapes.size() << std::endl;
std::cout << "# of materials : " << materials.size() << std::endl;

///1. 获取各种材质和纹理
{
    for (int i = 0; i < materials.size(); i++) {
        Material* m = new Material();
        tinyobj::material_t tm = materials[i];
        string name = tm.name;
        if (name.size()) {
            m->name = name;
        }
        m->ambient.r = tm.ambient[0];
        m->ambient.g = tm.ambient[1];
        m->ambient.b = tm.ambient[2];
        m->diffuse.r = tm.diffuse[0];
        m->diffuse.g = tm.diffuse[1];
        m->diffuse.b = tm.diffuse[2];
        m->specular.r = tm.specular[0];
        m->specular.g = tm.specular[1];
        m->specular.b = tm.specular[2];
        m->transmittance.r = tm.transmittance[0];
        m->transmittance.g = tm.transmittance[1];
        m->transmittance.b = tm.transmittance[2];
        m->emission.r = tm.emission[0];
        m->emission.g = tm.emission[1];
        m->emission.b = tm.emission[2];
        m->shininess = tm.shininess;
        m->ior = tm.ior;
        m->dissolve = tm.dissolve;
        m->illum = tm.illum;
        m->pad0 = tm.pad0;
        m->ambient_tex_id = -1;
        m->diffuse_tex_id = -1;
        m->specular_tex_id = -1;
        m->specular_highlight_tex_id = -1;
        m->bump_tex_id = -1;
        m->displacement_tex_id = -1;
        m->alpha_tex_id = -1;
        m->ambient_texname = "";
        m->diffuse_texname = "";
        m->specular_texname = "";
        m->specular_highlight_texname = "";
        m->bump_texname = "";
        m->displacement_texname = "";
        m->alpha_texname = "";
        if (tm.ambient_texname.size()) {
        }
        if (tm.diffuse_texname.size()) {
        }
        if (tm.specular_texname.size()) {
        }
        if (tm.specular_highlight_texname.size()) {
        }
        if (tm.bump_texname.size()) {
        }
        if (tm.displacement_texname.size()) {
        }
        if (tm.alpha_texname.size()) {
        }
        this->materials.push_back(m);
    }
}

/// 2.顶点数据
{
    // For each shape 遍历每一个部分
    for (size_t i = 0; i < shapes.size(); i++) {
        // 这部分的名称
        printf("shape[%ld].name = %s\n", static_cast<long>(i),
               shapes[i].name.c_str());
        // 网格的点数
        printf("Size of shape[%ld].mesh.indices: %lu\n", static_cast<long>(i),
               static_cast<unsigned long>(shapes[i].mesh.indices.size()));
        //printf("Size of shape[%ld].path.indices: %lu\n", static_cast<long>(i),static_cast<unsigned long>(shapes[i].path.indices.size()));
        //assert(shapes[i].mesh.num_face_vertices.size() == shapes[i].mesh.material_ids.size());
        //assert(shapes[i].mesh.num_face_vertices.size() == shapes[i].mesh.smoothing_group_ids.size());
        printf("shape[%ld].num_faces: %lu\n", static_cast<long>(i),
               static_cast<unsigned long>(shapes[i].mesh.num_face_vertices.size()));

        Model* model = new Model(); // 每一部分的模型数据
        // 顶点数量  = face的数量x3
        model->mesh_num = shapes[i].mesh.num_face_vertices.size() * 3;
        // 开辟空间
        Vertex *mesh_data = new Vertex[model->mesh_num];
        size_t index_offset = 0;
        // For each face
        for (size_t f = 0; f < shapes[i].mesh.num_face_vertices.size(); f++) {
            size_t fnum = shapes[i].mesh.num_face_vertices[f];
            // 获得所索引下标
            tinyobj::index_t idx;
            int vertex_index[3];
            int normal_index[3];
            int texcoord_index[3];

            for (size_t v = 0; v < fnum; v++) {
                idx = shapes[i].mesh.indices[index_offset + v];
                vertex_index[v] = idx.vertex_index;
                texcoord_index[v] = idx.texcoord_index;
                normal_index[v] = idx.normal_index;
            }

            for (size_t v = 0; v < fnum; v++) {
                // v
                mesh_data[index_offset + v].pos.x = attrib.vertices[(vertex_index[v]) * 3 + 0];
                mesh_data[index_offset + v].pos.y = attrib.vertices[(vertex_index[v]) * 3 + 1];
                mesh_data[index_offset + v].pos.z = attrib.vertices[(vertex_index[v]) * 3 + 2];
                mesh_data[index_offset + v].pos.w = 1.0f;
                // vt
                mesh_data[index_offset + v].tc.u = attrib.texcoords[texcoord_index[v] * 2 + 0];
                mesh_data[index_offset + v].tc.v = attrib.texcoords[texcoord_index[v] * 2 + 1];
                // vn
                mesh_data[index_offset + v].normal.x = attrib.normals[normal_index[v] * 3 + 0];
                mesh_data[index_offset + v].normal.y = attrib.normals[normal_index[v] * 3 + 1];
                mesh_data[index_offset + v].normal.z = attrib.normals[normal_index[v] * 3 + 2];
                mesh_data[index_offset + v].normal.w = 1.0f;
                // color
                mesh_data[index_offset + v].color.r = 1.0f;
                mesh_data[index_offset + v].color.g = 1.0f;
                mesh_data[index_offset + v].color.b = 1.0f;
                mesh_data[index_offset + v].color.a = 1.0f;
            }
            // 偏移
            index_offset += fnum;
        }
        model->mesh = mesh_data;
        models.push_back(model);
    }
}
```