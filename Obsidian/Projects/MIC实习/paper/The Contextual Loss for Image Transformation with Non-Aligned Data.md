# 概述
## 背景
- Image Transformation
	- Semantic style transfer
	- Single-image animation
	- ……
- Non-aligned Data
- 使用feed forward network，需要为此提供合适的loss function
## 问题
- Pixel-to-pixel：基于图像对其的前提，在现实情境中往往不满足对其的要求
- Global Loss function：以Gram loss function为代表，可以成功捕获texture与style信息，但是由于其全局的特性，会把全局的图像特性添加到整个生成图片中（怎么理解）
- 现有检测Contextual Similarity 的方法，导数没有意义，因此无法用于神经网络的训练（直接查找y中每个patch在x所有patch中最接近的一个，结果二值化，对y中每个patch，x中只有一个为1，因此导数无意义）
- GAN：大量的运算时间与算力需求
## 贡献
- 设计了一种loss function（Contextual Loss），可以用于测量两张图片之间的Contextul Loss，对空间信息具有鲁棒性，并且不具有Global特性，可以基于语意进行feature的迁移。可以用于feed forward network 的训练中
# 具体工作
- 设计一种测量Contextual Similarity 的loss function，下图是对Contextual similarity 的表示，每张图片都可以看成一系列点集（每个点是一个patch）![[Pasted image 20221111202937.png]]
- 以上图片，计算两个点之间的distance采用以下方法
	- $d_{ij}$表示$x_i$与$y_j$（两张图之间的两个patch）之间的Cosine Distance。           ![[Pasted image 20221124205750.png]]          
	- 计算$\tilde d_{ij}$ ，对距离进行normalize                                         ![[Pasted image 20221111203814.png]]
	- 将distance转化为similarity                                            ![[Pasted image 20221111203909.png]]
	- 对similarity进行标准化，定义patch之间的Contextual similarity                           ![[Pasted image 20221111210147.png]]   
	- 对于每个patch，计算最大的$CX_{ij}$，求和，计算得到两张图片的Contextual Similarity![[Pasted image 20221111211430.png]]
	- 将图片进行卷积，对于各个层级输出的feature map，取log计算contextual loss![[Pasted image 20221111212114.png]]
# 实验
## 定性实验
- 对一个确定的高斯分布和另外一个变化的高斯分布测量contextual similarity![[Pasted image 20221111215123.png]]
- Noise Pic与不对齐的target一同输入，进行图像还原，分别使用L1 loss和CX Loss![[Pasted image 20221111215216.png]]
- 风格迁移，可以发现GAN由于其Global特性，会将一些全局的style全部应用到生成的image中，比如第二排人像发绿，第三排狗头发白。CNNMRF比较保留local信息，但是效果稍微差于Contextual Loss![[Pasted image 20221111215610.png]]
# 启发
- Global与Local信息，patch大小的确定与效果非常相关。patch过小，相当于逐像素比较，会对空间变化过于敏感。但patch过大，会丢失local的信息。可以的想法：
	- 不同的卷积层输出分别包含了低级和高级的信息，当层级较低时，patch可以适当大以忽略几何的轻微变化；层级较高时，需要将patch减小（此时感受到的可能是一些语意信息，需要保持一致性）
# 疑问
- what is so-called “global”：
	- 过大的感受野导致网络更多感知到图像中的全局信息，对于一些局部的语意信息，网络可能忽略（关键在于确定patch的大小？），例子是人在草坪上，如果只感知global的contextual信息，人的位置发生变化（比如到天上去），此时发生了语意变化，但是整张图片的contextual没有太大的变化。