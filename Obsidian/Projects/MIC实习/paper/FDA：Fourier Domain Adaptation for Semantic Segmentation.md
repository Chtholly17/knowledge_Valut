[[storage/9FXA2S2Q/FDA：Fourier Domain Adaptation for Semantic Segmentation]]
## 概述
- UDA（Unsupervised Domain Adaption）
- semantic segmention：将图像中的每个像素分配一个类标签的过程（也称为语义类）
- moyivation：
	- l**ow-level spectrum (amplitude) can vary signiﬁcantly without affecting the perception of high-level semantics**.低阶的频域信息，在高阶语义信息没有发生改变的前提下，可能发生非常大的变化（这几个像素是人还是车，不应该受光照等信息的影响）
## 问题
- 语义分割问题中，一个域的图像（synthesis image for example）可能很容易大量获得，而另一个域的图像（real world image）可能无法大量采集
- 目前SOTA方法非常复杂，需要优化Adversarial loss
## 贡献
- 提出一个简单的UDA方法，通过交换不同domain之间的低频成分实现。在semantic segmention问题上进行了验证，效果良好（仅在语义分割问题上的表现来看，甚至超过了当前的SOTA方法）
## 具体工作
- FDA：
	- 中央低频部分方形mask![[Pasted image 20230323103719.png]]
	- FDA的定义如下：对Amplitude进行mask，对phase保留![[Pasted image 20230323103739.png]]
- FDA for Semantic Segmentation
	- 对source domain的每张图片，进行一次FDA（从target domain中随机选一张），然后送入网络推理，计算结果和GT之间的差别![[Pasted image 20230323104054.png]]
	- Target Domain中，是一个半监督问题，需要限制产生结果的熵![[Pasted image 20230323104146.png]]
	-  使用不同的FDA中的beta值，训练一系列模型，然后在target domain中求arg max![[Pasted image 20230323104825.png]]
	- Overall loss![[Pasted image 20230323104835.png]]
## 实验
- 定性实验![[Pasted image 20230323104934.png]]
## 启发
- 傅立叶分解中低频的幅值成分：有什么含义？
	- 幅值：某种style、texture特征
	- 幅值的低频：某些低频的texture？应该是图像的整体背景颜色、整体风格等。
- 本文通过训练在不同domain中模型对幅值低频信息的Adaption，可以用一种简单的方法实现UDA。
- 说明：在不同domain之间的shift，很可能就**存在于低频的风格信息中**，高频的语义信息之间应该不会有太大的差别