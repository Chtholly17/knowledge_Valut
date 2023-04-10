# 概述
## 背景
- six kinds of semantics in CNNs, i.e. objects, parts, scenes, textures, materials, and colors.
	- 前两种具有特定的形状，描述物体的某个部分。后四种没有清晰的轮廓，描述某种纹理特性
	- 低层级的filter往往描述简单的纹理，而高层级的filter可能表示物体的组成（几何变化）
## 问题
- 传统的CNN高层级的一个filter可能表示图像中物体的多个组成部分，造成可解释性的下降。![[Pasted image 20230113012348.png]]
## 贡献
- 本文研究的目标为：
	- 通过简单地修改一个CNN以提升其可解释性（每个高层级的filter代表物体的一个部分），类似的方法可以用在不同结构的多种CNN上
	- 不需要对物体的每个部分进行任何人工标注
	- 不改变输出层的loss function，可以使用与传统CNN相同的方法训练
	- 可能会降低网络的鉴别能力（discrimination power），但可以将这种降低约束在一定范围内
- 提出了一种简单有效的loss，这个loss作用在网络中每个filter产生的feature map上，使网络的每个filter代表物体的一部分，这种loss具有以下的特点：
	- 每个filter编码了某一类物体的某个特定部分，不应该编码多种不同类型的某个部分。（low entropy of inter-category activations）
	- 每个filter只编码了一个部分，只能被一个物体的一个部分激活，不能代表物体的多个部分。（low entropy of spatial distributions of neural activations）
	- 假设在图像中多个位置重复出现的形状描述了低级的纹理（比如形状和边缘），而并非描述了高阶的物体组成部分。
		- 意思是重复出现的形状可能只是代表了低级纹理的重复，在高级语义的角度看它们可能是不同的
		- 比如左眼和右眼，多次出现，但只说明低级纹理（颜色、边缘上）可能重复，而作为高级的context信息来看，它们是对称的但不是相同的，因此它们可能会被不同的filter代表。
# 具体工作
- Framework![[Pasted image 20230113112104.png]]
	- 红色与绿色箭头分别代表forward和backward
	- 在forward过程中，每个interpretable filter会被分配一个mask
		- 假设relu过后，feature map的spatial size为n\*n，为空间中的每个点分配一个template，每个template是一个n\*n的矩阵，描述了理想情况下，当以feature map中对应的这个点为中心时（也就是通过这个滤波器得到的特征图中这个位置的点数值最大）所具有的分布（对于不同的图像CNN可能会选择不同的templates），如下图所示：![[Pasted image 20230113114307.png]]
			- $\tau$和$\beta$是某个常数，$||.||_1$是L-1距离![[Pasted image 20230113154137.png]]
		- 为每个滤波器输出的feature map进行mask操作，实际上就是找到feature map中数值最大的那个点，将feature map和最大值点对应的template进行逐元素相乘。得到masked feature map，这样的mask操作可以支持梯度回传![[Pasted image 20230113114558.png]]
	- backward过程中，希望每个filter对应某种类型图片中的一个part
		- 每种类型有自己对应的n2种templates
		- 假设图片属于n2种templates对应的类型，则应该在这n2中选择一种templates对应计算mask
		- 如果不属于对应的类型，应该选择T-作为对应的template，注意这是在backward过程中的处理方法（在forward中，一定是从n2种templates种选择，无法选择T-），T-的公式如下，所有点均为一个负常数$\tau$![[Pasted image 20230113154531.png]]
		- 假设backward过程中选择的tempate（有n2+1种）为T，x为这个filter输出的feature map，则使用它们二者之间的互信息（muttual information）的负数作为loss![[Pasted image 20230113153352.png]]
			- $P(T_{\mu})=\frac{\alpha}{n^2},P(T^{-})=1-\alpha$，alpha是一个常数
			- ![[Pasted image 20230113153804.png]]
			- ![[Pasted image 20230113153822.png]]
			- tr是矩阵的trace
		- 以上loss的计算实际上就是评价x与T（某个templates）之间的信息差别
	- 学习过程
		- backward过程中，每个filter都会接受到梯度，按以下公式计算：![[Pasted image 20230113160050.png]]
		- lambda是权重参数
		- $L(\hat{y}_k,y_k^*)$是网络中正常计算出的Loss
		- 具体实现：![[Pasted image 20230113160442.png]]
		- 为每个filter分配category，用于计算以上的梯度，使用以下公式，实际上就是找到最能激发这个filter的种类![[Pasted image 20230113160813.png]]
- 为什么上述的$Loss_f$可以看作是Low inter- category entropy和Low spatial entropy的，首先以上$Loss_f$可以等价于以下公式![[Pasted image 20230113163540.png]]
	- Low inter- category entropy：![[Pasted image 20230113163725.png]]
		- 所有的T+都可以看作是一个代表某一种类c的label![[Pasted image 20230113164211.png]]
		- 用一个负template T-代表其他类，使filter必须只对于一类图片有反应，可以用输出的特征图f来判断输入的图片是否属于可以激活该filter的类别
	- Low spatial entropy![[Pasted image 20230113164041.png]]
		- 每个filter只应该被图片中的一个part激活，而不应该被空间中多个位置激活![[Pasted image 20230113164113.png]]
# 实验
## 定性结果
- 高层级卷积层filter感受野的可视化![[Pasted image 20230113165056.png]]
- 使用顶层卷积层所有卷积层所有的filter可视化成为的heat map![[Pasted image 20230113165220.png]]