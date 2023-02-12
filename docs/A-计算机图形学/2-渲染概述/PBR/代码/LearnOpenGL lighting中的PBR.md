教程：[光照 - LearnOpenGL CN (learnopengl-cn.github.io)](https://learnopengl-cn.github.io/07%20PBR/02%20Lighting/)


## 源代码
### 1.1.pbr.vs
```glsl
#version 330 core
layout (location = 0) in vec3 aPos;         //点位置（模型坐标系）
layout (location = 1) in vec3 aNormal;      //法向量（模型坐标系）
layout (location = 2) in vec2 aTexCoords;   //UV

out vec2 TexCoords;
out vec3 WorldPos;
out vec3 Normal;

uniform mat4 projection;    //投影矩阵
uniform mat4 view;          //视图矩阵
uniform mat4 model;         //模型矩阵

void main()
{
    TexCoords = aTexCoords;                     //UV
    WorldPos = vec3(model * vec4(aPos, 1.0));   //点位置（世界坐标系）
    Normal = mat3(model) * aNormal;             //法向量（世界坐标系）
    //注：mat3(model)是为了去掉平移值，只做旋转与缩放等变换，因为它是向量，不是坐标点

    gl_Position =  projection * view * vec4(WorldPos, 1.0); //点位置（标准化设备坐标）
}
```

### 1.1.pbr.fs
```glsl
#version 330 core
out vec4 FragColor;
in vec2 TexCoords;  //UV
in vec3 WorldPos;   //片元位置（世界坐标）
in vec3 Normal;     //法向量（世界坐标）

//材质参数
// material parameters
uniform vec3 albedo;        //反照率
uniform float metallic;     //金属度
uniform float roughness;    //粗糙度
uniform float ao;           //环境光遮蔽

// lights
uniform vec3 lightPositions[4]; //灯光位置
uniform vec3 lightColors[4];    //灯光颜色（实际上不是颜色，而是光的辐射通量，用RGB三色编码来表示）

uniform vec3 camPos;    //摄像机位置（世界坐标系）

const float PI = 3.14159265359;
// ----------------------------------------------------------------------------
//GGX的贡献度
float DistributionGGX(vec3 N, vec3 H, float roughness)
{
    float a = roughness*roughness;
    float a2 = a*a;
    float NdotH = max(dot(N, H), 0.0);
    float NdotH2 = NdotH*NdotH;

    float nom   = a2;
    float denom = (NdotH2 * (a2 - 1.0) + 1.0);
    denom = PI * denom * denom;

    return nom / denom;
}
// ----------------------------------------------------------------------------
//计算Schlick-GGX
float GeometrySchlickGGX(float NdotV, float roughness)
{
    //k是roughness的重映射，本代码使用直接光照
    float r = (roughness + 1.0);
    float k = (r*r) / 8.0;

    float nom   = NdotV;
    float denom = NdotV * (1.0 - k) + k;

    return nom / denom;
}
// ----------------------------------------------------------------------------
//几何函数Smith
// 微平面间相互遮蔽的比率，这种相互遮蔽会损耗光线的能量
float GeometrySmith(vec3 N, vec3 V, vec3 L, float roughness)
{
    float NdotV = max(dot(N, V), 0.0);
    float NdotL = max(dot(N, L), 0.0);
    float ggx2 = GeometrySchlickGGX(NdotV, roughness);
    float ggx1 = GeometrySchlickGGX(NdotL, roughness);

    return ggx1 * ggx2;
}
// ----------------------------------------------------------------------------
//菲涅尔函数
// 被反射的光线对比光线被折射的部分所占的比率
vec3 fresnelSchlick(float cosTheta, vec3 F0)
{
    return F0 + (1.0 - F0) * pow(clamp(1.0 - cosTheta, 0.0, 1.0), 5.0);
}

// ----------------------------------------------------------------------------
void main()
{
    vec3 N = normalize(Normal);             //法向量（世界坐标系）
    vec3 V = normalize(camPos - WorldPos);  //光线出射方向、视线方向（指向摄像机，世界坐标系）

    // calculate reflectance at normal incidence; if dia-electric (like plastic) use F0 
    // of 0.04 and if it's a metal, use the albedo color as F0 (metallic workflow)    
    //计算正常入射时的反射率
    //1. 如果介质电(如塑料)使用F0为0.04
    //2. 如果是金属，F0=albedo(金属工作流程)
    vec3 F0 = vec3(0.04); 
    F0 = mix(F0, albedo, metallic);

    //反射率方程
    // reflectance equation
    vec3 Lo = vec3(0.0);
    for(int i = 0; i < 4; ++i) 
    {
        // calculate per-light radiance
        vec3 L = normalize(lightPositions[i] - WorldPos);       //光线入射方向（指向光源，世界坐标系）
        vec3 H = normalize(V + L);                              //半程向量
        float distance = length(lightPositions[i] - WorldPos);  //与光源的距离
        float attenuation = 1.0 / (distance * distance);        //光源衰减程度（距离越远，衰减越严重）
        vec3 radiance = lightColors[i] * attenuation;           //辐射率，光源打在此片元上的总能量

        // Cook-Torrance BRDF
        float NDF = DistributionGGX(N, H, roughness);                   //法线方程  
        float G   = GeometrySmith(N, V, L, roughness);                  //几何函数
        vec3 F    = fresnelSchlick(clamp(dot(H, V), 0.0, 1.0), F0);     //菲涅尔函数
           
        vec3 numerator    = NDF * G * F; 
        float denominator = 4.0 * max(dot(N, V), 0.0) * max(dot(N, L), 0.0) + 0.0001; // + 0.0001 to prevent divide by zero
        vec3 specular = numerator / denominator;    //Cook-Torrance BRDF 镜面反射部分的值
        
        // kS is equal to Fresnel
        //kS(镜面反射系数) = 菲涅尔项
        vec3 kS = F;
        // for energy conservation, the diffuse and specular light can't
        // be above 1.0 (unless the surface emits light); to preserve this
        // relationship the diffuse component (kD) should equal 1.0 - kS.
        //为了能量守恒，kD+kS应等于1
        //因此kD(漫反射)
        vec3 kD = vec3(1.0) - kS;
        // multiply kD by the inverse metalness such that only non-metals 
        // have diffuse lighting, or a linear blend if partly metal (pure metals
        // have no diffuse light).
        //非金属才有漫反射（金属没有漫反射），因此kD需乘上逆金属度
        kD *= 1.0 - metallic;

        // scale light by NdotL
        float NdotL = max(dot(N, L), 0.0);        

        // add to outgoing radiance Lo
        //lambert = albedo / PI
        Lo += (kD * albedo / PI + specular) * radiance * NdotL;  // note that we already multiplied the BRDF by the Fresnel (kS) so we won't multiply by kS again
    }   
    
    // ambient lighting (note that the next IBL tutorial will replace 
    // this ambient lighting with environment lighting).
    //环境照明，在下一个IBL中教程中，将用环境光来实现
    vec3 ambient = vec3(0.03) * albedo * ao;

    //最终的颜色
    vec3 color = ambient + Lo;

    // HDR tonemapping
    //HDR技术之tone mapping
    color = color / (color + vec3(1.0));
    // gamma correct
    //gamma矫正
    color = pow(color, vec3(1.0/2.2)); 

    FragColor = vec4(color, 1.0);
}
```


### main.cpp
```cpp
//...
int main()
{
	// build and compile shaders
    // -------------------------
    Shader shader("1.1.pbr.vs", "1.1.pbr.fs");

    shader.use();
    shader.setVec3("albedo", 0.5f, 0.0f, 0.0f);
    shader.setFloat("ao", 1.0f);

    // lights
    // ------
    glm::vec3 lightPositions[] = {
        glm::vec3(-10.0f,  10.0f, 10.0f),
        glm::vec3( 10.0f,  10.0f, 10.0f),
        glm::vec3(-10.0f, -10.0f, 10.0f),
        glm::vec3( 10.0f, -10.0f, 10.0f),
    };
    glm::vec3 lightColors[] = {
        glm::vec3(300.0f, 300.0f, 300.0f),
        glm::vec3(300.0f, 300.0f, 300.0f),
        glm::vec3(300.0f, 300.0f, 300.0f),
        glm::vec3(300.0f, 300.0f, 300.0f)
    };
    int nrRows    = 7;
    int nrColumns = 7;

	while (!glfwWindowShouldClose(window))
	{
		for (int row = 0; row < nrRows; ++row) 
        {
            shader.setFloat("metallic", (float)row / (float)nrRows);
            for (int col = 0; col < nrColumns; ++col) 
            {
                // we clamp the roughness to 0.05 - 1.0 as perfectly smooth surfaces (roughness of 0.0) tend to look a bit off
                // on direct lighting.
                shader.setFloat("roughness", glm::clamp((float)col / (float)nrColumns, 0.05f, 1.0f));

				//...
                renderSphere();
            }
        }
    }
	

	return 0;
}
```