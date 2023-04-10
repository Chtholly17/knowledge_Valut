[(23条消息) Visual Studio开源库集成器Vcpkg全教程--利用Vcpkg轻松集成开源第三方库_Achilles的博客-CSDN博客](https://blog.csdn.net/cjmqas/article/details/79282847)
- 平台：windows10专业版+VS 2022 community
- git clone从gitee上克隆项目的镜像![[Pasted image 20221120151719.png]]
- 进入项目根目录并运行批处理指令![[Pasted image 20221120152606.png]]
- 集成到全局![[Pasted image 20221120152633.png]]
- 可能需要在vs的powershell中再次运行一次集成到全局![[Pasted image 20221120163314.png]]
接下来对项目进行如下的设置：![[Pasted image 20221120163359.png]]
- 注意：设置installed Directory是必要的
## vcpkg换源
- 添加系统变量，并编辑对应的值![[Pasted image 20221120160446.png]]
## vcpkg ninja编译错误
- 类似以下的错误，ninja.exe -v ERROR CODE 1![[Pasted image 20221120184456.png]]
解决方法：cmake出错，编译的路径中不可以有特殊字符，之前的路径中包括c++，改为cpp即可[Failed to compile regex - 简书 (jianshu.io)](http://events.jianshu.io/p/73dd7ee26047)
