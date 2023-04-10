## 1 Texture的添加
* 设置纹理分布，需要在vbo中操作（textCoord是一个vec2
```cpp
// calculate triangle vertices
	for (int i = 0; i <= prec; i++) {
		for (int j = 0; j <= prec; j++) {
			float y = (float)cos(toRadians(180.0f - i * 180.0f / prec));
			float x = -(float)cos(toRadians(j * 360.0f / prec)) * (float)abs(cos(asin(y)));
			float z = (float)sin(toRadians(j * 360.0f / prec)) * (float)abs(cos(asin(y)));
			vertices[i * (prec + 1) + j] = glm::vec3(x, y, z);
			texCoords[i * (prec + 1) + j] = glm::vec2(((float)j / prec), ((float)i / prec));
			normals[i * (prec + 1) + j] = glm::vec3(x, y, z);
		}
	}
```
```cpp
int numIndices = mySphere.getNumIndices();
	for (int i = 0; i < numIndices; i++) {		// 把每一个点上的坐标（x,y,z），纹理坐标（s，t），法向量(a,b,c)存储进对应数组
		pvalues.push_back((vert[ind[i]]).x);
		pvalues.push_back((vert[ind[i]]).y);
		pvalues.push_back((vert[ind[i]]).z);
		tvalues.push_back((tex[ind[i]]).s);
		tvalues.push_back((tex[ind[i]]).t);
		nvalues.push_back((norm[ind[i]]).x);
		nvalues.push_back((norm[ind[i]]).y);
		nvalues.push_back((norm[ind[i]]).z);
	}

	glGenVertexArrays(1, vao);
	glBindVertexArray(vao[0]);
	glGenBuffers(numVBOs, vbo);
	// put the vertices into buffer #0  第一个是顶点放入缓存器中
	glBindBuffer(GL_ARRAY_BUFFER, vbo[0]);
	glBufferData(GL_ARRAY_BUFFER, pvalues.size() * 4, &pvalues[0], GL_STATIC_DRAW);
	// put the texture coordinates into buffer #1  第二个是将纹理坐标放入缓存器中
	glBindBuffer(GL_ARRAY_BUFFER, vbo[1]);
	glBufferData(GL_ARRAY_BUFFER, tvalues.size() * 4, &tvalues[0], GL_STATIC_DRAW);
	// put the normals into buffer #2   第三个是将法向量放入缓存器中
	glBindBuffer(GL_ARRAY_BUFFER, vbo[2]);
	glBufferData(GL_ARRAY_BUFFER, nvalues.size() * 4, &nvalues[0], GL_STATIC_DRAW);
```
* 使用learn openGL中的loadTexture函数，获取TexureID
	* 相关依赖为stb_image.h
```cpp
//load textures
	vector<string> enum_str = { "Sun.jpg","Mercury.jpg","Venus.jpg","Earth.jpg","Moon.jpg",
								"Mars.jpg","Jupiter.jpg","Saturn.jpg","Uranus.jpg","Neptune.jpg","Galaxy.jpg" };
	for (int i = 0; i < STARS_NUM; i++) {
		string path = "texture/" + enum_str[i];
		unsigned int tmp = loadTexture(path.c_str());
		textureMap.push_back(tmp);
	}
```
* 使用纹理之前，激活一个纹理即可
```cpp
//Bind texture
    glActiveTexture(GL_TEXTURE0);
```
* 选中拥有对应纹理的vbo，
```cpp
// 纹理
	glBindBuffer(GL_ARRAY_BUFFER, vbo[1]);
	glVertexAttribPointer(1, 2, GL_FLOAT, GL_FALSE, 0, 0);
	glEnableVertexAttribArray(1);
	bindTexture();
```
* 使用texture ID，绑定纹理
```cpp
void Star::bindTexture() {
	int textureID = textureMap[(int)type];
	glBindTexture(GL_TEXTURE_2D, textureID);
}
```
* draw
```cpp
// adjust OpenGL settings and draw model
	glEnable(GL_DEPTH_TEST);
	glDepthFunc(GL_LEQUAL);
	glFrontFace(GL_CCW);// 锥体的三角形是逆时针的面认为是正方向
	glDrawArrays(GL_TRIANGLES, 0, mySphere.getNumIndices());
```
## 2 奇怪的问题
* 实际上并不是语法问题
![[Pasted image 20221029224555.png]]
## 3 glVertexAttribPointer的使用
```cpp
//normal
	glBindBuffer(GL_ARRAY_BUFFER, vbo[2]);
	glVertexAttribPointer(2, 3, GL_FLOAT, GL_FALSE, 0, 0);
	glEnableVertexAttribArray(2);
```
-   第一个参数指定我们要配置的顶点属性。还记得我们在顶点着色器中使用`layout(location = 0)`定义了position顶点属性的位置值(Location)吗？它可以把顶点属性的位置值设置为`0`。因为我们希望把数据传递到这一个顶点属性中，所以这里我们传入`0`。

-   第二个参数指定顶点属性的大小。顶点属性是一个`vec3`，它由3个值组成，所以大小是3。

-   第三个参数指定数据的类型，这里是GL_FLOAT(GLSL中`vec*`都是由浮点数值组成的)。

-   下个参数定义我们是否希望数据被标准化(Normalize)。如果我们设置为GL_TRUE，所有数据都会被映射到0（对于有符号型signed数据是-1）到1之间。我们把它设置为GL_FALSE。

-   第五个参数叫做步长(Stride)，它告诉我们在连续的顶点属性组之间的间隔。由于下个组位置数据在3个`float`之后，我们把步长设置为`3 * sizeof(float)`。要注意的是由于我们知道这个数组是紧密排列的（在两个顶点属性之间没有空隙）我们也可以设置为0来让OpenGL决定具体步长是多少（只有当数值是紧密排列时才可用）。一旦我们有更多的顶点属性，我们就必须更小心地定义每个顶点属性之间的间隔，我们在后面会看到更多的例子（译注: 这个参数的意思简单说就是从这个属性第二次出现的地方到整个数组0位置之间有多少字节）。

-   最后一个参数的类型是`void*`，所以需要我们进行这个奇怪的强制类型转换。它表示位置数据在缓冲中起始位置的偏移量(Offset)。由于位置数据在数组的开头，所以这里是0。我们会在后面详细解释这个参数。
注意：glEnableVertexAttribArray应该与glVertexAttribPointer配套使用，每次绑定之后使用Enable激活
![[Pasted image 20221030155000.png]]