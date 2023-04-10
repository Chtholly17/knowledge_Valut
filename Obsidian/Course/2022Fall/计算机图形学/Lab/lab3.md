本次lab中，对于三种不同的material，我分别使用了不同的方式进行渲染。下面分别进行介绍
## light
对于本身就是光源的情况，直接返回其表面纹理的颜色，与光栅化采用相似的处理方式
```cpp
MatType type = rec.mat_ptr->type;
//return albedo as final color directly for light source
if (type == MatType::LIGHT) {
	finalColor = rec.mat_ptr->albedo;
}
```
## glass
对于玻璃材质的物体，必须要考虑其反射光线与折射光线，分别使用递归的方式进行计算
- 反射
首先计算出反射光的方向，对产生的反射光线递归地调用trace函数进行追踪，最终将产生的结果乘上反射强度。
```cpp
//reflect light
//normalize vector to calculate Direction of reflected light
glm::vec3 rayDir = glm::normalize(ray.dir);
glm::vec3 normal = glm::normalize(rec.normal);
glm::vec3 reflectDir = glm::reflect(rayDir, normal);
//generate reflected ray
Ray rr = Ray(rec.p, reflectDir);
color reflectColor = trace(rr, scene, depth + 1);
finalColor = reflectColor * reflect_atten;
```
- 折射
对于折射光线首先需要根据折射率计算出折射光线的方向，需要注意如果折射角过大折射光将不可见，此时不需要考虑折射。计算出折射光线之后同样使用trace递归计算。需要注意的是，此处需要考虑入射光线的方向，如果是从表面的反向入射，需要将折射率求倒数。
```cpp
//refract light
float refractIndex = rec.front_face ? rec.mat_ptr->refraction_ratio : 1.0f / rec.mat_ptr->refraction_ratio;
float cos = glm::dot(-rayDir, normal);
float sin = sqrt(1 - cos * cos);
float rsin = sin * refractIndex;
//refract happen
if(rsin < 1.0f) {
	// generate refract light
	glm::vec3 refractDir = glm::refract(rayDir, normal, refractIndex);
	Ray tr = Ray(rec.p, refractDir);
	color refractColor = trace(tr, scene, depth + 1);
	finalColor += refractColor * refract_atten;
}
```
## diffuse
对于既不是光源，也不是玻璃的物体，采用与光栅化方法中类似的处理方式，对于环境中所有光源，使用phong model分别计算ambient、diffuse、specular，并乘上对应的强度系数之后求和。以下是具体的计算过程
```cpp
for (std::shared_ptr<hittable> l : scene->lights) {
	shared_ptr<Sphere> light = std::static_pointer_cast<Sphere>(l);

	color lightColor = light->mat_ptr->albedo;
	color materialColor = rec.mat_ptr->albedo;
	// ambient
	color ambient = ambientIndex * lightColor * materialColor;
	// diffuse
	glm::vec3 norm = glm::normalize(rec.normal);
	glm::vec3 lightDir = glm::normalize(light->center - rec.p);
	float diff = std::max(glm::dot(norm, lightDir), 0.0f);
	color diffuse = (diffuseIndex * materialColor) * (diff * lightColor);
	// specular
	glm::vec3 viewDir = glm::normalize(scene->camera->origin - rec.p);
	glm::vec3 reflectDir = glm::reflect(-lightDir, norm);
	float spec = powf(std::max(glm::dot(viewDir, reflectDir), 0.0f), shininess);
	color specular = (specularIndex * materialColor) * (spec * lightColor);
```
需要注意的是，为了产生shadow，需要产生一个shadowRay，并采用类似的方法判断其相交情况。如果shadowRay直接与光源相交则不会产生阴影，否则处于阴影之中。如果处在阴影中，只需要考虑其环境光产生的颜色即可，不考虑speculate和diffuse。
```cpp
	finalColor = ambient;
	// if in shadow
	Ray shadowRay = Ray(rec.p, lightDir);
	hit_record srec;
	scene->hit(shadowRay, 0.001f, infinity, srec);
	// not in a shadow
	if (srec.object == light) {
		finalColor += specular + diffuse;
	}
}
```
## 最终效果
![[out.jpg]]