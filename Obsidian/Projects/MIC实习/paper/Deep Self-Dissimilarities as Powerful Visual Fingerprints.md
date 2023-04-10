# 概述
## 背景
- high level semantics and low level textural information of feature
- 测量feature 的 difference
- per-element similarity measures
	- including LPIPS and perceputal loss, 本质上是对feature map进行逐像素的计算（l2等方式）
- internal distribution of image
	- including contextual loss and PDL, 在patch的层面上研究distribution的区别
- 图片的不同scale的deep feature具有很大的区别，以下的例子中，不同解析度的同样图片使相同的VGG产生了不同的分类结果![[Pasted image 20221206172256.png]]
## 贡献
- DSD: deep self-dissmilarity，可以捕获图片各个scale的distribution的差异，适合用作图片的fingerprint，用作FR与NR IQA或者用于restoration 的 loss
# 具体工作
- DSD的定义
	- 使用Gram metric衡量一个scale中i、j两个channel之间的关联![[Pasted image 20221206185238.png]]
		- k表示spatial上的一个position，l表示网络的l层输出的feature map，x表示输入的图片
		- 实际上就是将同一scale中不同channel的两张feature逐像素相乘后求和，WH分别为这一层spatial size的H和W
		- $G_l(x)$是一个矩阵，其中的第(i,j)个元素就由上述公式计算出
	- DSD的定义如下![[Pasted image 20221206185634.png]]
		- l为网络的第l层输出的scale进行计算
		- alpha为下采样的倍率
	- 类似的，对原图（具有RGB三个通道也可以定义类似的DSD，如下![[Pasted image 20221206190407.png]]
		- 首先，将原图分为尺寸为n\*n的多个patch
		- 每个patch第i、j个像素相乘，然后所有patch的结果求和，W、H是原图的Width和Height
		- $G_{RGB}(x)$是一个matrix，其中第i行第j个元素按照以上公式计算
- 可以使用DSD作为FR IQA指标![[Pasted image 20221206200312.png]] ![[Pasted image 20221206200555.png]]
	 - 可以认为是一种跨Scale的deep feature distribution difference
	 - DSD
# 实验
## 定性实验
- 对feature map上和原图上分别计算DSD、l1距离，得如下结果![[Pasted image 20221206190910.png]]
	- 蓝：从数据集中随机取图片，计算对应层$G_l(x)$之间的l1距离
	- 红：从数据集与对应图片属于同一类的图片中随机取，还是计算l1
	- 绿：计算每张图片的DSD，采用alpha=4
	- 我们可以观察出以下现象：
		- 在feature map层面，在浅层的feature上，原图和downscale之后的图片非常相似，但深层feature上，原图和downscale之后图片的DSD甚至超过了完全无关的随机图片的$G_l(x)$之间的l1距离
		- 在原图上，当patch越大，三个值都越小。可能说明是大的patch不同于逐像素的比较方法，同一数据集中不同图片之间的统计分布层面上的信息差距被减少。同时，DSD一直最小
- DSD的" visualising"![[Pasted image 20221206192358.png]]
	- 可以看到，具有低DSD评分的图片往往是变化比较平滑的自然风景，而DSD高的则具有非常高频变化的texture
- 进行style transfer，分别采用最小化DSD（下方）和style loss（上方），采用DSD增加了更锐利、细节的texture，说明DSD更适合用作图像的fingerprint![[Pasted image 20221206195009.png]]
- 使用DSD计算SR任务的Loss![[Pasted image 20221206202830.png]]
	- Lrec为l1距离，Lper为perceptual distance![[Pasted image 20221206202929.png]]
- DSD用作deblurring![[Pasted image 20221206203430.png]] ![[Pasted image 20221206203440.png]] 
## 定量实验
- DSD用作FR IQA metric![[Pasted image 20221206201243.png]]
	- 深层feature计算的DSD效果很好
- DSD用作NR IQA metric，训练一个网络获取y的degradation-free version![[Pasted image 20221206202442.png]] ![[Pasted image 20221206202454.png]]
- 将DSD用作SR任务，三个指标分别为PSNR、LPIPS、NIQE![[Pasted image 20221206203258.png]] ![[Pasted image 20221206203348.png]]
	- 注意NIQE越低越好
- DSD用作deblurring![[Pasted image 20221206203509.png]]
# 启发
- DSD作为一种能很好表现图像结构信息的fingerprint，可以认为是跨channel地对该scale的feature进行了融合。可以考虑将其作为附加项直接加入到我们的损失函数中来，以添加跨channel的信息感知能力吗？
- 为什么DSD可以包含图像的特征，是否意味着这种特征存在于high-resolution的图片与down scale图片之间的差距中，或者说down scale实际上是在保存distribution大致不变的情况下丢失了一些东西。正是这些丢失的深层特征导致了downscale的结果与原图观感的差异，而DSD恰好捕捉了这种差异。有趣的是，我们发现这种差异足以成为图片的fingerprint，说明这种差异可能就是图像texture层面某些信息的表示。
- 实际上DSD可能表示了高频信息的丢失，Gram系数可能是对高频信息的提取，而同一图片的downscale结果可能丢失的就是高频信息。DSD是高频信息的fingerPrint