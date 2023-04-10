## 1、创建源文件
首先需要创建main.cpp
## 2 、添加include文件

-   右键项目 `OpenGLExercise` ，在弹出的选项中，单击 `属性`
-   点击 `C/C++ —> 常规 —> 附加包含目录 —> 编辑`  
![](https://pic1.zhimg.com/80/v2-34c282a64018d445fb7693feae52fe9c_720w.webp)
* 点击添加头文件
	* D:/opengl/include中添加glm、glad、KHR的头文件![[Pasted image 20221027213514.png]]
	* D:/opengl/glfw-3.3.8.bin.WIN32/include中添加GLFW头文件![[Pasted image 20221027213555.png]]
# 3、添加lib文件
-   点击 `链接器 —> 常规 —> 附加包含目录 —> 编辑`  
![](https://pic2.zhimg.com/80/v2-09cc8769222a6dde39e0d8e3449e1a4d_720w.webp)
* 添加GLFW即可，需要注意版本，使用VS2019对应的版本，路径为：D:/opengl/glfw-3.3.8.bin.WIN32/lib-vc2019![[Pasted image 20221027213932.png]]
# 4、添加库依赖项
-   点击 `链接器 —> 输入 —> 附加依赖项 —> 编辑`  输入opengl32.lib与
glfw3.lib
![](https://pic3.zhimg.com/80/v2-3e7c17f3fa87a3c5ad7ecfb449743afe_720w.webp)
# 5、加入glad.c
将glad.c加入项目的源文件中，路径为D:/opengl/src![[Pasted image 20221027214541.png]]
# 6、解决冲突
点击属性，找到“编辑”
![](https://pic4.zhimg.com/80/v2-19ca3270133c019f85acff8c0662ba27_720w.webp)
输入如下：

```text
MSVCRT.lib
LIBCMT.lib
```
最后，点击一下右下角的应用，再点击确定。
![](https://pic4.zhimg.com/80/v2-86968535797faac012afb08ab512c58b_720w.webp)
