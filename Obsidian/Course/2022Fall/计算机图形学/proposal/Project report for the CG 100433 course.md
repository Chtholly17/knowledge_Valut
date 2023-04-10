
## Project Title
台球室
## Team member
2053195 姜垒
2054087 孙睿
2054088 王子安
2051988 马嘉
2053764 吴俊成
2053635 任柯睿
## Abstract
我们希望采用多种技术渲染一个尽可能真实的台球厅，并在场景中添加与台球的交互。
## Motivation
在当前的CG技术中，PBR技术与光线追踪技术，分别在渲染贴近真实的复杂材质以及模拟复杂反射折射方面取得了较为良好的成果。在本项目中，我们的目标是依托台球室场景进行组合交互，动态展现物体在运动时的渲染效果。我们希望借助这个项目，在直观认识两个技术优势的同时，理解光线追踪与传统流水线渲染方式之间的异同，同时提升图形学变成水平以及较复杂项目的设计实现水平。![[屏幕截图 2022-12-10 2344212.png]]

## The goal of the project
1. 展示较为逼真的台球室全局场景。
2. 使用PBR技术让物体表面更加符合物理实际。
3. 借助物理引擎实现台球击打和碰撞。
4. 使用光追技术实现较为真实的光照效果。
5. 良好的人机交互体验。
## The scope of the project
- 场景中仅有台球可交互
- 不实现除台球与台球之间、台球与台球桌之间之外的物体碰撞检测
- 光追仅展示渲染效果，渲染帧数过低无法添加交互
- 追求对场景的真实再现
- 追求对物理现象的真实模拟
## Involved CG techniques
- Obj格式模型的导入和使用
- 阴影渲染
- PBR
- Bullet物理引擎
- 光线追踪
## Project Contents
1. 精美的渲染效果
我们使用了PBR和光追多种技术进行渲染，实现了非常逼真的渲染效果。
2. 逼真的交互
我们使用Bullet物理引擎并自行设计了交互逻辑，实现了真实、流畅、有趣的交互体验。
## Implementation
### 模型导入
1. 模型加载库ASSIMP
我们目前使用的模型加载库是ASSIMP，其支持对多种格式的模型文件的解析，并将其转化为ASSIMP中特有的SCENE对象，我们对SCENE对象下的各类数据进行读取并存入对应的结构体，就可以获得各个顶点的坐标、索引、纹理、材质等，最终再通过图形管线进行模型的渲染。
2. OBJ文件与MTL文件
OBJ格式的文件提供顶点坐标、索引、纹理坐标，不包含动画、材质特性、贴图路径、动力学、粒子等信息。
OBJ文件需要MTL辅助文件来声明贴图路径、贴图位置等表面贴图信息。
3. 遇到的问题
**问题1: MTL文件中链接纹理图片的方式较为特殊，对于处理PBR材质没有专门的链接接口。**

![[Pasted image 20221230215011.png]]
解决方法：不通过ASSIMP库加载MTL中链接的纹理图片，自己编写函数读取。MTL文件中的一个纹理，通过map_Ka绑定ambient环境光贴图,map_Kd绑定diffuse贴图（ao贴图），map_Ks绑定高光贴图（roughness）
**问题2: 单个模型文件可能拥有多种纹理，需要链接多个mtl材质，这时候一个Model需要对应多种PBR纹理（比如收银台上，桌子用的是一组PBR，收银台用的是另一组PBR，但是它们共享同一个OBJ和MTL文件）。这会在后续的着色器编写中带来麻烦（使用的PBR组数不确定，计算规则也会发生变化）。**

解决方法：将单个模型文件不同材质的部分分为两个使用单组PBR纹理的OBJ文件进行导入，可以使用统一的着色器。（采用了这种方式）
4. 导入方式
不通过MTL链接纹理图片，直接编写函数，读取同一文件夹下的五张PBR贴图。读取OBJ模型读入原始模型，再与读取的PBR贴图进行链接，最终导入正确显示的模型。
5. 类实现
分为两个模型类：staticModel类、dynamicModel类
（1） staticModel类
这个类用于加载静态模型，在后续的交互过程中不需要进行设置，在最初的模型导入就将位置进行固定。
```cpp
class staticModel {
    Model model;
    Shader shader;
    Shader simpleDepthShader;//“ı”∞
public:
    bool is_wall;
    bool is_shadow;
    staticModel(const char* filePath, const char* vsPath, const char* fsPath, bool isShadow = true, bool isWall = false)
        :model(filePath), shader(vsPath, fsPath), simpleDepthShader(
            "shader/shadow.vs",
            "shader/shadow.fs",
            "shader/shadow.gs"
        ) {
        is_wall = isWall;
        is_shadow = isShadow;
    }
    void setShader() {
        // don't forget to enable shader before setting uniforms
        shader.use();
        shader.setInt("albedoMap", 0);
        shader.setInt("normalMap", 1);
        shader.setInt("metallicMap", 2);
        shader.setInt("roughnessMap", 3);
        shader.setInt("aoMap", 4);
        shader.setInt("depthMap", 5);

        // view/projection transformations
        glm::mat4 projection = glm::perspective(glm::radians(camera.Zoom), (float)SCR_WIDTH / (float)SCR_HEIGHT, 0.1f, 100.0f);
        glm::mat4 view = camera.GetViewMatrix();
        shader.setMat4("projection", projection);
        shader.setMat4("view", view);
        shader.setVec3("camPos", camera.Position);

        // render the loaded model
        glm::mat4 model = glm::mat4(1.0f);
        
        shader.setMat4("model", model);
        for (unsigned int i = 0; i < sizeof(lightPositions) / sizeof(lightPositions[0]); ++i) {
            shader.setVec3("lightPositions[" + std::to_string(i) + "]", lightPositions[i]);
            shader.setVec3("lightColors[" + std::to_string(i) + "]", lightColors[i]);
            //shader.setBool("lightShadows[" + std::to_string(i) + "]", lightShadows[i]);
        }
        shader.setInt("shadows", is_wall); // enable/disable shadows by pressing 'SPACE'
        shader.setFloat("far_plane", far_plane);
        if (is_wall)
        {
            shader.setInt("reverse_normals", 1); // A small little hack to invert normals when drawing cube from the inside so lighting still works.
        }
        else
        {
            shader.setInt("reverse_normals", 0); // A small little hack to invert normals when drawing cube from the inside so lighting still works.
        }
    }

    void setsimpleDepthShader(std::vector<glm::mat4>& shadowTransforms, glm::vec3 lightPos)
    {
        simpleDepthShader.use();
        for (unsigned int i = 0; i < 6; ++i)
            simpleDepthShader.setMat4("shadowMatrices[" + std::to_string(i) + "]", shadowTransforms[i]);
        simpleDepthShader.setFloat("far_plane", far_plane);
        simpleDepthShader.setVec3("lightPos", lightPos);
        glm::mat4 model = glm::mat4(1.0f);
        simpleDepthShader.setMat4("model", model);
    }

    void Draw() {
        model.Draw();
    }
};
```
（2） dynamicModel类
这个类用于加载动态模型（台球杆和台球），在后续的交互过程中需要进行动画设置，预留出接口进行模型变换。
```cpp
class dynamicModel {
    Model model;
    Shader shader;
    glm::vec3 scaleVec;
    glm::vec3 translateVec;
    glm::vec3 rotateVec;
    float rotateAngle;
    glm::mat4 modelMatirx;
    Shader simpleDepthShader;//“ı”∞
public:
    bool is_shadow;
    dynamicModel(const char* filePath, const char* vsPath, const char* fsPath, bool isShadow = true)
        :model(filePath), shader(vsPath, fsPath), simpleDepthShader(
            "shader/shadow.vs",
            "shader/shadow.fs",
            "shader/shadow.gs"
        ) {
        scaleVec = glm::vec3(1.0f, 1.0f, 1.0f);
        translateVec = glm::vec3(0.0f, 0.0f, 0.0f);
        rotateVec = glm::vec3(1.0f, 1.0f, 1.0f);
        rotateAngle = 0.0f;
        is_shadow = isShadow;
    }

    void setScaleVec(glm::vec3 setVec)
    {
        scaleVec = setVec;
    }
    
    void setRotateAngle(float setAngle)
    {
        rotateAngle = setAngle;
    }

    void setRotateVec(glm::vec3 setVec)
    {
        rotateVec = setVec;
    }
    
    void setTranslateVec(glm::vec3 setVec)
    {
        translateVec = setVec;
    }

    void Rotate(glm::vec3 vec, float angle) {
        // std::cout<<"rotate called"<<std::endl;
        modelMatirx = glm::rotate(modelMatirx, glm::radians(angle), vec);
        // std::cout<<"rotate:"<<std::endl;
        // std::cout<<vec.x<<" "<<vec.y<<" "<<vec.z<<std::endl;
        // std::cout<<angle<<std::endl;
    }

    void Trans(glm::vec3 vec) {
        // std::cout<<"translate called"<<std::endl;
        modelMatirx = glm::translate(modelMatirx, vec);
    }

    void setShader() {
        // don't forget to enable shader before setting uniforms
        shader.use();
        shader.setInt("albedoMap", 0);
        shader.setInt("normalMap", 1);
        shader.setInt("metallicMap", 2);
        shader.setInt("roughnessMap", 3);
        shader.setInt("aoMap", 4);
        shader.setInt("depthMap", 5);

        // view/projection transformations
        glm::mat4 projection = glm::perspective(glm::radians(camera.Zoom), (float)SCR_WIDTH / (float)SCR_HEIGHT, 0.1f, 100.0f);
        glm::mat4 view = camera.GetViewMatrix();
        shader.setMat4("projection", projection);
        shader.setMat4("view", view);
        shader.setVec3("camPos", camera.Position);

        // render the loaded model
        modelMatirx = glm::mat4(1.0f);

        shader.setInt("shadows", false); // enable/disable shadows by pressing 'SPACE'
        shader.setFloat("far_plane", far_plane);
        shader.setInt("reverse_normals", 0); // A small little hack to invert normals when drawing cube from the inside so lighting still works.

        //modelMatirx = glm::translate(modelMatirx, translateVec);
        modelMatirx = glm::scale(modelMatirx, scaleVec);
        //modelMatirx = glm::rotate(modelMatirx, glm::radians(rotateAngle), rotateVec);

    }

    void setModel() {
        shader.setMat4("model", modelMatirx);
        for (unsigned int i = 0; i < sizeof(lightPositions) / sizeof(lightPositions[0]); ++i) {
            glm::vec3 newPos = lightPositions[i];
            shader.setVec3("lightPositions[" + std::to_string(i) + "]", newPos);
            shader.setVec3("lightColors[" + std::to_string(i) + "]", lightColors[i]);
        }

    }

    void setsimpleDepthShader(std::vector<glm::mat4>& shadowTransforms, glm::vec3 lightPos)
    {
        simpleDepthShader.use();
        for (unsigned int i = 0; i < 6; ++i)
            simpleDepthShader.setMat4("shadowMatrices[" + std::to_string(i) + "]", shadowTransforms[i]);
        simpleDepthShader.setFloat("far_plane", far_plane);
        simpleDepthShader.setVec3("lightPos", lightPos);
        simpleDepthShader.setMat4("model", modelMatirx);
    }

    void Draw() {
        model.Draw();
    }

};
```
### 物理引擎
**1.**  **物理引擎的选择**
为了实现桌球游戏中的较为精密的物理效果，我们需要采用物理引擎来进行实现。而支持c++语言的物理引擎种类繁多，我们在进行相关实现之前对市面上流行的c++物理引擎进行了一定的调研。其中，project chrono适用于数量极多的物体间的物理效果模拟，对于我们只有数十个带有物理效果的物体的简单场景来说过于复杂。另外，市面上另一种较为热门且易上手的物理引擎box2d由于其二维物理引擎的特点，因此较难满足我们对于台球复杂滚动以及碰撞响应的模拟要求。因此我们最后采用的是bullet物理引擎，其API设计较为简单，同时有相关的论坛以及文档等，便于初学者上手使用。同时bullet引擎是c++编写的开源3D引擎，具备一定的性能表现以及模拟精度，能够较好地满足本次实践的应用需求。
**2.**  **物理引擎的封装**
为了降低物理引擎的使用难度，同时提供较为简单的API接口以便和项目其他部分进行对接，首先对物理引擎进行封装。分析本项目对于物理引擎的需求，其目的在于，通过对部分具有物理效果的对象施加如速度、冲量的控制，通过物理引擎进行相关物理现象的模拟后返回物体的坐标以及旋转方向角度等位姿信息，具体流程如下图所示。![[Pasted image 20221230222310.png]]
根据上述要求，设计物理引擎的相关API，需要提供包括注册模型信息、冲量速度等的控制以及获取物体位姿信息的相关接口。其具体UML类图如下图所示。![[Pasted image 20221230222326.png]]
其中，BulletUtil类实现了进行相关刚体世界、碰撞列表、重力因素等的初始化操作。同时定义相关物理碰撞物体基类CollisionObject并提供最为基础的控制函数，用以控制相关刚体的速度、冲量等，并获取位置坐标等相关信息。通过继承该基类，我们拥有CollisionSphere碰撞球类、CollisionBox碰撞盒类以及CollisionPool碰撞球桌类。
在实现上，CollisionSphere以及CollisionBox类都采用了bullet引擎内置的规则形状，分别通过球心半径以及盒体中心与长宽高的方式进行初始化。而CollisionPool在本次应用中用于刻画具有不规则形状的台球桌。借助bullet手册中如下图所示的API选择指导，对于一个不规则的静态台球桌形状，我们采用了btConvexHullShape的方式进行构建。具体如何获取模型信息已经在上文中有较为清晰的阐述，在此就不多做赘述。在本类中，通过复用在模型导入阶段编写的Model类，以便读入模型形状供物理引擎进行相关模拟。![[Pasted image 20221231144513.png]]
**3.**  **实现细节**
为了获得更为贴近真实的桌球效果，我们在实现过程中对物理引擎里各个物体的相关参数进行了大量的调整。首先在桌球的旋转上，由于游戏控制方式是通过鼠标来确定球杆的击球点位的，因此实际上较难瞄准桌球的正中心。在大力出杆时尝尝会导致桌球旋转的幅度过大，造成运动轨迹不规则的情况。因此我们首先增大了桌球的转动惯量，使得它的转动幅度减小，在运动上更为自然。同时增大了球体的滚动摩擦以及转动摩擦，以便更好地模拟台球在台泥上因自身旋转而导致的路径偏移。同时我们略微增大了台球桌的弹性恢复系数，使得桌球在接触台球边缘后能够有一定程度反弹，以模拟真实的台球击打效果。但是这样做就造成当球杆大力向下击打时，球体会产生激烈的跳动，甚至飞出桌外。为解决这个问题，我们手动限制了台球在垂直方向上的最大速度以及最大高度，以保证在能够完成诸如跳球等行为的同时，能够有更加自然的物理表现。
### PBR
**1. PBR简介**
PBR(Physically Based Rendering)，是指基于物理的渲染。通过使用PBR渲染，我们可以使用一种更符合物理学规律的方式来模拟光线，这种渲染方式与我们原来的Phong或者Blinn-Phong光照算法相比总体上看起来要更真实一些。![[Pasted image 20221230214656.png]]
ALBEDO：表面纹理，指定表面颜色或者基础反射率。
NORMAL：法线贴图，法线贴图使我们可以逐片段的指定独特的法线，来为表面制造出起伏不平的假象。
METALLIC：金属度，设定纹素的金属质地。
ROUGHNESS：粗糙度。样得来的粗糙度数值会影响一个表面的微平面统计学上的取向度。一个比较粗糙的表面会得到更宽阔更模糊的镜面反射（高光），而一个比较光滑的表面则会得到集中而清晰的镜面反射。
AO：环境光遮蔽，将光线较难逃逸出来的暗色边缘指定出来。
**2. PBR的实现**
依据渲染方程，在fragment shader中实现PBR模型$$L_o(p,\omega_o) = \int\limits_{\Omega} (k_d\frac{c}{\pi} + k_s\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)}) L_i(p,\omega_i) n \cdot \omega_i  d\omega_i$$
当前只需要遍历所有有贡献的光源，计算其对辐照度的贡献即可，当后续需要将环境光考虑在内的情况下，必须使用积分去计算可能从任何一个方向上入射的光线。对于每一个光源，需要计算完整的Cook-Torrance specular BRDF项：
$$\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)}$$
- F项：使用fresnelSchlick方程计算，返回以物体表面光线被反射的百分比。
```cpp
vec3 fresnelSchlick(float cosTheta, vec3 F0) { return F0 + (1.0 - F0) * pow(1.0 - cosTheta, 5.0); }
```
- D项：法线分布函数（Normal Distribution Function）
```cpp
float DistributionGGX(vec3 N, vec3 H, float roughness) 
{ 
	float a = roughness*roughness;
	float a2 = a*a; 
	float NdotH = max(dot(N, H), 0.0); 
	float NdotH2 = NdotH*NdotH; 
	float nom = a2; 
	float denom = (NdotH2 * (a2 - 1.0) + 1.0); 
	denom = PI * denom * denom; 
	return nom / denom; 
}
```
- G项：几何遮蔽函数
```cpp
float GeometrySchlickGGX(float NdotV, float roughness) 
{ 
	float r = (roughness + 1.0); 
	float k = (r*r) / 8.0; 
	float nom = NdotV; 
	float denom = NdotV * (1.0 - k) + k; 
	return nom / denom; 
} 

float GeometrySmith(vec3 N, vec3 V, vec3 L, float roughness) 
{ 
	float NdotV = max(dot(N, V), 0.0);
	float NdotL = max(dot(N, L), 0.0); 
	float ggx2 = GeometrySchlickGGX(NdotV, roughness); 
	float ggx1 = GeometrySchlickGGX(NdotL, roughness);
	return ggx1 * ggx2; 
}
```
- 计算Cook-Torrance BRDF
```cpp
vec3 nominator = NDF * G * F; 
float denominator = 4.0 * max(dot(N, V), 0.0) * max(dot(N, L), 0.0) + 0.001;
vec3 specular = nominator / denominator;
```
- 之后为每个光源分配贡献值，fresnelSchlick方程给出了Ks，可以从Ks计算出折射的比值KD,我们可以看作ks表示光能中被反射的能量的比例， 而剩下的光能会被折射， 比值即为kD。更进一步来说，因为金属不会折射光线，因此不会有漫反射。所以如果表面是金属的，我们会把系数kD变为0。
```cpp
vec3 kS = F;
vec3 kD = vec3(1.0) - kS;
kD *= 1.0 - metallic;
```
- 最后需要进行伽马矫正，完整的PBR着色器如下
```cpp
#version 330 core

out vec4 FragColor;
in vec2 TexCoords;
in vec3 WorldPos;
in vec3 Normal;

// material parameters
uniform sampler2D albedoMap;
uniform sampler2D normalMap;
uniform sampler2D metallicMap;
uniform sampler2D roughnessMap;
uniform sampler2D aoMap;

// lights
uniform vec3 lightPositions[4];
uniform vec3 lightColors[4];

uniform vec3 camPos;

const float PI = 3.14159265359;
// ----------------------------------------------------------------------------
// Easy trick to get tangent-normals to world-space to keep PBR code simplified.
// Don't worry if you don't get what's going on; you generally want to do normal 
// mapping the usual way for performance anways; I do plan make a note of this 
// technique somewhere later in the normal mapping tutorial.
vec3 getNormalFromMap()
{
    vec3 tangentNormal = texture(normalMap, TexCoords).xyz * 2.0 - 1.0;

    vec3 Q1  = dFdx(WorldPos);
    vec3 Q2  = dFdy(WorldPos);
    vec2 st1 = dFdx(TexCoords);
    vec2 st2 = dFdy(TexCoords);

    vec3 N   = normalize(Normal);
    vec3 T  = normalize(Q1*st2.t - Q2*st1.t);
    vec3 B  = -normalize(cross(N, T));
    mat3 TBN = mat3(T, B, N);

    return normalize(TBN * tangentNormal);
}
// ----------------------------------------------------------------------------
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
float GeometrySchlickGGX(float NdotV, float roughness)
{
    float r = (roughness + 1.0);
    float k = (r*r) / 8.0;

    float nom   = NdotV;
    float denom = NdotV * (1.0 - k) + k;

    return nom / denom;
}
// ----------------------------------------------------------------------------
float GeometrySmith(vec3 N, vec3 V, vec3 L, float roughness)
{
    float NdotV = max(dot(N, V), 0.0);
    float NdotL = max(dot(N, L), 0.0);
    float ggx2 = GeometrySchlickGGX(NdotV, roughness);
    float ggx1 = GeometrySchlickGGX(NdotL, roughness);

    return ggx1 * ggx2;
}
// ----------------------------------------------------------------------------
vec3 fresnelSchlick(float cosTheta, vec3 F0)
{
    return F0 + (1.0 - F0) * pow(clamp(1.0 - cosTheta, 0.0, 1.0), 5.0);
}
// ----------------------------------------------------------------------------
void main()
{		
    vec3 albedo     = pow(texture(albedoMap, TexCoords).rgb, vec3(2.2));
    float metallic  = texture(metallicMap, TexCoords).r;
    float roughness = texture(roughnessMap, TexCoords).r;
    float ao        = texture(aoMap, TexCoords).r;

    vec3 N = getNormalFromMap();
    vec3 V = normalize(camPos - WorldPos);

    // calculate reflectance at normal incidence; if dia-electric (like plastic) use F0 
    // of 0.04 and if it's a metal, use the albedo color as F0 (metallic workflow)    
    vec3 F0 = vec3(0.04); 
    F0 = mix(F0, albedo, metallic);

    // reflectance equation
    vec3 Lo = vec3(0.0);
    for(int i = 0; i < 4; ++i) 
    {
        // calculate per-light radiance
        vec3 L = normalize(lightPositions[i] - WorldPos);
        vec3 H = normalize(V + L);
        float distance = length(lightPositions[i] - WorldPos);
        float attenuation = 1.0 / (distance * distance);
        vec3 radiance = lightColors[i] * attenuation;

        // Cook-Torrance BRDF
        float NDF = DistributionGGX(N, H, roughness);   
        float G   = GeometrySmith(N, V, L, roughness);      
        vec3 F    = fresnelSchlick(max(dot(H, V), 0.0), F0);
           
        vec3 numerator    = NDF * G * F; 
        float denominator = 4.0 * max(dot(N, V), 0.0) * max(dot(N, L), 0.0) + 0.0001; // + 0.0001 to prevent divide by zero
        vec3 specular = numerator / denominator;
        
        // kS is equal to Fresnel
        vec3 kS = F;
        // for energy conservation, the diffuse and specular light can't
        // be above 1.0 (unless the surface emits light); to preserve this
        // relationship the diffuse component (kD) should equal 1.0 - kS.
        vec3 kD = vec3(1.0) - kS;
        // multiply kD by the inverse metalness such that only non-metals 
        // have diffuse lighting, or a linear blend if partly metal (pure metals
        // have no diffuse light).
        kD *= 1.0 - metallic;	  

        // scale light by NdotL
        float NdotL = max(dot(N, L), 0.0);        

        // add to outgoing radiance Lo
        Lo += (kD * albedo / PI + specular) * radiance * NdotL;  // note that we already multiplied the BRDF by the Fresnel (kS) so we won't multiply by kS again
    }   
    
    // ambient lighting (note that the next IBL tutorial will replace 
    // this ambient lighting with environment lighting).
    vec3 ambient = vec3(0.03) * albedo * ao;
    
    vec3 color = ambient + Lo;

    // HDR tonemapping
    color = color / (color + vec3(1.0));
    // gamma correct
    color = pow(color, vec3(1.0/2.2)); 

    FragColor = vec4(color, 1.0);
}
```
### 阴影
由于深度贴图的分辨率固定，因此在多个片段对应于一个纹理像素时，多个片段会从深度贴图的同一个深度值进行采样，因此这几个片段会产生锯齿边。为解决这个问题，我们小组选择使用柔和阴影的策略，其核心思想是从深度贴图中多次采样，每一次采样的纹理坐标都稍有不同。每个独立的样本可能在也可能不再阴影中。所有的次生结果接着结合在一起，进行平均化。
$C=C*(1-\frac{1}{n}\sum_{i=1}^{n}f(shadaw))$
```cpp
float ShadowCalculation(vec3 fragPos,vec3 lightPos)
{
    // get vector between fragment position and light position
    vec3 fragToLight = fragPos - lightPos;
    float currentDepth = length(fragToLight);
    float shadow = 0.0;
    float bias = 0.15;
    int samples = 20;
    float viewDistance = length(camPos - fragPos);
    float diskRadius = (1.0 + (viewDistance / far_plane)) / 25.0;
    for(int i = 0; i < samples; ++i)
    {
        float closestDepth = texture(depthMap, fragToLight + gridSamplingDisk[i] * diskRadius).r;
        closestDepth *= far_plane;   // undo mapping [0;1]
        if(currentDepth - bias > closestDepth)
            shadow += 1.0;
    }
    shadow /= float(samples); 
        
    return shadow;
}
```
### 光线追踪
- BVH 树：使用 BVH 树可以加速光线和三角形求交的过程。BVH 树类似于平衡二叉树，将三角形分成两个包围盒，在理想的情况下，如果每次都能对半分并且光线只和其中的一个盒子有交，时间复杂度能从 $O(n)$ 降至 $O(\log n)$ 。
- SAH（Surface Area Heuristic）优化：在 BVH 树的基础上更为合理地划分三角形，具体的在划分的时候计算一个预计的求交代价 $cost=pn$ 其中 $p$ 为击中的概率，$n$ 为三角形的个数，在这里将 AABB 包围盒的面积作为启发。使用 SAH 优化进行建树能有效解决划分不均的问题。
- Disney Principle's BRDF：Disney 的 BRDF 参数直观，对艺术家友好，且有很强的表现力。公式如下：
$$f(l,v)= diffuse+\frac{D(\theta_h)F(\theta_d)G(\theta_l,\theta_v)}{4\cos{\theta_l}\cos{\theta_v}}$$
其中$l$是入射光的反向量，$v$是观察方向的向量，$h$是半角向量（或者微表面法线）。diffuse 是一个未定式，通常使用足够简单的 Lambert 漫反射模型，但是 Disney BRDF 选择了更为复杂的表达式。$\frac{D(\theta_h)F(\theta_d)G(\theta_l,\theta_v)}{4\cos{\theta_l}\cos{\theta_v}}$表示的是高光，由$D,F,G$组成，$D$是微表面法线分布项，$F$是菲涅尔项，$G$是几何遮蔽自阴影项。Disney Principle Parameters 给出了 1 个 color 参数和 10 个 scalar 参数。
- baseColor：表面颜色，通常作为纹理采样的颜色
- subsurface：次表面散射参数，用于将次表面散射和漫反射插值
- metallic：金属度，决定了漫反射的比例
- specular：控制镜面反射强度
- specularTint：控制镜面反射的颜色，根据此参数在 baseColor 和 vec3(1)之间插值
- roughness：粗糙度
- anisotropic：各向异性参数。
- sheen：模拟织物布料边缘的透光
- sheenTint：类似 specularTint，控制植物高光颜色在 baseColor 和 vec3(1)之间插值
- clearcoat：清漆强度，模拟粗糙物体表面的光滑涂层
- clearcoatGloss：清漆的粗糙度或者光泽程度。
- 低差异序列：通常情况下，我们使用蒙特卡洛采样，来对困难积分的值进行估计。蒙特卡洛积分是无偏的，但是积分域的某些角落，在采样次数较少的时候难以达到，采样次数上去了还是可以采到的。而对于低差异序列，非常少量的采样就可以均匀照顾到区域，因此低差异序列在采样次数低的时候能够更快收敛。使用的低差异序列是 Sobal 序列。Sobol 序列的每一个维度都是由底数为 2 的 radical inversion 组成，但每一个维度的 radical inversion 都有各自不同的矩阵$C$。
$X_i:=(\Phi_{2,C_1}(i),\ldots,\Phi_{2,C_n}(i))$
因为完全以 2 为底数，所以 Sobol 序列的生成可以直接使用 bit 位操作实现 radical inversion，非常高效。Sobol 序列的分布具有不仅均匀，同时又不需要预先确定样本的数量或者将样本储存起来，并可以根据需要生成无限个样本，非常适合 progressive 的采样。
- 重要性采样：概率统计告诉我们，相比于各个位置均匀采样，如果在函数值大的地方多采样几次就能够更好的拟合，这个策略叫做重要性采样。如果随机变量的概率密度函数$pdf(x)$ 能够正比于被估计函数的函数值$f(x)$，那么估计的结果最好。渲染方程的蒙特卡洛采样形式：
$$
L_o(p,w_o)\approx L_e(p,w_o) + \frac{1}{N}\sum_{i=1}^N\frac{L_i(p,w_i)f_r(p,w_i,w_o)(n\cdot w_i)}{p(w_i)}
$$
最佳的采样$pdf$应该正比于被积函数$f(x)$，但在实际中往往是不可能的。因为$pdf(x)=cf(x)$需要提前知道所有关于$f(x)$的信息，就比如你想算$L_i$其实本身就是另一个积分问题了。 虽然没有办法得到最优的$pdf$，但是另一个被广泛采纳的策略是让$pdf$​与被积函数中的一部分成正比。那么我们就有了两个重要性采样的对象：$(n\cdot w_i)$和 BRDF。对于 Disney BRDF 的 diffuse 项，我们可以近似地认为$$F_d=\frac{baseColor}{\pi}cos(w_i)$$那么也对 cosine 重要性采样也相当于对漫反射采样。
#### 1.光线追踪总体实现思路
与光栅化不同，光线追踪使用基于物理的实现方法，通过模拟光在介质中的真实表现来输出足以逼近现实的图像。由于从光源发射出发光线可以有无数条，并有无数个方向，从光源出发有无数条到达相机的路线。直接正向地从光源开始模拟光的传播变得不是很现实，我们将光线传播的路径反过来，从摄像机开始发射光线，试图找出到达光源的可能路径。这边是本次光线追踪的核心技术——路径追踪技术，在光线传播的过程中不断采样，不断积累颜色，最终返回积累的颜色作为屏幕中某个像素点的颜色。
在光线追踪过程中，使用渲染方程渲染某个像素的颜色，并使用上述所说明的技术对光线追踪的过程进行优化，最终完成整个光线追踪的结果。![[Pasted image 20221231141837.png]]
具体实现过程为：
1.布置画布
2.三角形数据传送到 shader
3.在 shader 中进行三角形求交
4.线性化 BVH 树
5.BVH 数据传送到 shader
6.和 AABB 盒子求交
7.非递归遍历 BVH 树
8.具体光线追踪过程
9.后处理部分
#### 2.GPU 的使用
本次光线追踪使用 GPU 进行加速，先将模型等数据读取到内存中，之后再以纹理的形式把这些数据传送到 GPU，然后在 shader 中对纹理进行采样就可以读出原来的数据。通过这种方法，我们可以把三角形数组，BVH 二叉树数组，材质数组等等先传递到 GPU 中，之后在 GPU 中进行光线追踪的计算，便可实现加速。
在这个过程中我们使用 OpenGL 中的 Buffer Texture，它允许我们直接将内存中的二进制数据搬运到显存中，然后通过 samplerBuffer 来访问。samplerBuffer 把 GPU 中的原始数据视为一维数组，可以通过下标直接索引数据，并且不会使用任何过滤器，这一特性也极大的方便了我们在 GPU 中对于数据的使用。
我们之前所学到的 BVH 树遍历时基于递归方法的，但是 GPU 与 CPU 不同，其没有栈的概念，所以为了迎合 GPU 的这一特点，我们使用了非递归方法对 BVH 树进行了遍历。
#### 3.实现细节
在这部分，我们简要说明一些除光线追踪最基本理论实现之外的一些具体实现细节。
##### 3.1 纹理贴图手动插值
与光栅化不同，光线追踪中的 shader 不会自动对纹理进行插值，所以我们需要手动插值把纹理颜色附着到物体上。
本次手动插值的具体实现是在计算光线和三角形求交的过程中求出交点对应的 uv 坐标，我们使用的插值方式是根据该点与三角形三个顶点分别形成的三个内部三角形所占面积的比进行插值。同时，这种方法也可以运用在计算该点的法线上。
##### 3.2 镜子实现
与光栅化相比，光线追踪在渲染阴影、反射、折射方面有着天然的优势。为了进一步展示本次光线追踪的效果，我们在台球桌中间添加了一面镜子，通过镜子反射影像展示光线追踪的技术优点。
镜子的特性主要体现在其材质参数中，我们通过对roughness,metallic,specular,baseColor,specularTint 等参数进行调整，最终在一个扁平的长方体模型上实现了镜子的效果，具体参数如下：
```C++
m.roughness = 0.00;
m.metallic = 0.8;
m.specular = 1.0;
m.baseColor = vec3(1, 1, 1);
m.specularTint = 1.0;
float len = 1.5;
readObj("resources/quad.obj", triangles, m, getTransformMatrix(vec3(0, 0, -90), vec3(1.0, -1.4, 0), vec3(len, 0.01, len)), false);

```
##### 3.3 算法外的加速技巧
尽管本次光线追踪已经使用了 BVH 树、SAH 优化等算法进行了加速，但是由于本次项目所用所有物体都为实际建模，渲染代价依旧很大，导致渲染速度仍不尽人意，最终我们采用了以下几个算法之外的方法对光追进行进一步加速
- 模型缩小：模型的表面积越大，渲染速度就越慢，因此我们把所有物体大小都缩小为了原来的十分之一，从而减小模型表面积，加速渲染。
- 模型坍缩：通过边坍缩的方法，减小模型的三角形个数，可以减小建立 BVH 树以及三角形求交的代价。在坍缩过程中，我们根据模型本身的复杂程度和特点，将其面数坍缩到了原来的 0.2-0.5，在尽可能保证模型形状不变的前提下，减少其面数。
- 缩小窗口：上面曾经提到，本次光线追踪是对屏幕窗口中的每一个像素颜色进行求解，缩小窗口也就减少了需要求解颜色的像素个数，从而减轻了渲染代价。我们将窗口大小缩小到 256*256，获得了不错的渲染速度，将 FPS 稳定在了 70 左右
## Result
#### PBR渲染效果
![[Pasted image 20221230222619.png]]
![[Pasted image 20221230222553.png]]
![[Pasted image 20221230222600.png]]
- 阴影
![[Pasted image 20221230222655.png]]
![[Pasted image 20221230222612.png]]
#### 光线追踪
1.  全景图
![[Pasted image 20221231142539.png]]
    光追结果展示图1：全景图
    可以看到两个光源分别发出的黄光和白光；立在台球桌上的镜子对球的反射。
2.  桌椅子
![[Pasted image 20221231142624.png]] 
    光追结果展示图2：沙发和桌子
![[Pasted image 20221231142636.png]]
    光追结果展示图3：吧台
    桌子和沙发在光源的照射下形成非常自然的影子。门由于经过了模型的简化和原始的模型相差较大，导致表面的纹理有些许紊乱。
3.  镜子
    ![[Pasted image 20221231142704.png]]
    光追结果展示图4：镜子
    可以清楚的显示镜子中对台球的反射并且对比光追结果展示图 1 可以发现不同角度观察下，镜子能反射到不同的物体，图 4 就将两个光源反射出来。
## Roles in group
- 孙睿：光追基础原理实现，将光栅化中的模型导入到光追中，调整模型参数。
- 姜垒：光追优化部分实现，简化模型，提高光追效率。
- 王子安：负责物理引擎的挑选、封装以及整合工作，参与游戏逻辑的设计以及部分模型的修改以及坐标计算等
- 马嘉：所有室内模型的建模，在流水线中导入模型，参与PBR部分的实现。
- 任柯睿：PBR、HDR贴图的实现，在流水线方法中实现阴影绘制、最终显示效果的调整。
- 吴俊成：PBR部分的实现、负责流水线方法中交互逻辑的设计和实现。
## Reference
\[1\]B. Burley, “Physically Based Shading at Disney,” p. 27.
\[2\]S. Joe and F. Y. Kuo, “Notes on generating Sobol sequences,” p. 3.
\[3\][https://blog.yiningkarlli.com/2015/02/multiple-importance-sampling.html](https://blog.yiningkarlli.com/2015/02/multiple-importance-sampling.html)
\[4\][https://www.pbr-book.org/3ed-2018/Monte_Carlo_Integration/Importance_Sampling#sec:importance-sampling](https://www.pbr-book.org/3ed-2018/Monte_Carlo_Integration/Importance_Sampling#sec:importance-sampling)
\[5\][https://zhuanlan.zhihu.com/p/20197323](https://zhuanlan.zhihu.com/p/20197323)
\[6\]bullet3/docs at master · bulletphysics/bullet3[EB/OL]. GitHub. /2022-12-30. https://github.com/bulletphysics/bullet3.
\[7\]Bullet Collision Detection & Physics Library: Class List[EB/OL]. /2022-12-30. https://pybullet.org/Bullet/BulletFull/annotated.html.
\[8\]https://learnopengl-cn.github.io