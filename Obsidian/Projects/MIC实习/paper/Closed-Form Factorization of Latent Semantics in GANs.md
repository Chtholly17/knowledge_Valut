# 概述
## 背景
- image editing
- semantic meaning of GAN in latent space
- supervised method: 在latent space大量随机采样，对随机生成的图片进行标注，训练一个latent space中的分类器
	- Generative Mechanism of GANS：
		- translate a code in latent space into an image in image space, 这一过程通常使用CNN实现，通过多层卷积进行逐步的转化。
		- 本文中主要关注直接作用在latent space上的转化，这一步通常可以被表示成一个afﬁne transformation，其中A、b是投影参数向量，y是投影的结果，维度为m![[Pasted image 20221216093814.png]]
	- 在GAN latent space上某个特定的方向进行线性的改动，可以更改图片某种特定的semantic，如下：![[Pasted image 20221216110157.png]]
	- 结合以上两种情况，假设在latent space上进行了某种修改，GAN generator第一步的运算将会如下述公式所示：![[Pasted image 20221216110515.png]]
		- 通过以上公式可以观察到，当给定一个latent space上的向量(z)以及一个direction(n)时，GAN generator的第一步运算与参数矩阵A之间具有很大的关系，因此本文尝试通过分解矩阵A从而确定latent space中重要的方向。
## 问题
- supervised method：
	- sampling 耗费巨大时间并且不稳定
	- 有时候分类器难以训练甚至无法训练
	- 人工标注消耗大量时间
## 贡献
- SeFa：提出一种在latent space上进行分析的方法，独立于模型与训练数据，可以提取出latent space中较为“重要”的方向。
# 具体工作
- 依据之前的分析，首先给出了计算latent space中“最重要”方向（每个latent space上的方向可以看作image space上的某种semantic）的公式：                                                              ![[Pasted image 20221216112120.png]]
	- 此公式的本质实际上就是找出，在使用参数矩阵A进行投影的前提下，哪个方向可以使运算结果发生最大的变化
- 假设需要寻找latent space中前k重要的direction，可以采用如下公式：![[Pasted image 20221216112352.png]]
	- 为了求解这个问题，引入Lagrange Multiplier，将问题转化为以下形式：![[Pasted image 20221216112641.png]]
	- 对每个ni求偏微分，可以得到以下方程：![[Pasted image 20221216112725.png]]
	- 综上所述：所有上述方程可能的解都是矩阵$A^TA$的一个奇异向量，用矩阵N的每一列作为与$A^TA$前k大奇异值对应的奇异向量。
- 以上算法在GAN上的实现：
	- PGGAN：卷积生成器的代表，将latent code先转化为feature，经过一系列卷积后投影到image space，针对这种GAN，SeFa主要学习从Latent Code 到 feature的转变。
	- styleGAN：每一层都需要将latent code映射到Style code并输入网络中，SeFa可以学习从latent space 到 style space的转变过程
	- BigGAN：latent code既在开始时输入，也在每一层都输入，因此可以看作以上两种GAN的结合。
# 实验
## 定性实验
- 每组图片中，中间的是原图，左右分别在SeFa找出的重要方向上朝不同方向移动之后的结果：![[Pasted image 20221216115337.png]]
- 不同DIrection上的数值调节直接改变生成图片中的某些语意信息![[Pasted image 20221216115543.png]]
- 在styleGAN的不同层上调节SeFa寻找到的Direction上的位移，可以发现，底层的网络更倾向于更改旋转（比较大的几何语义变化），中层网络倾向于改变形状（可能介于几何与Texture之间），高层网络主要倾向于改变颜色。![[Pasted image 20221216115727.png]]
- 与GAN space（上一篇文章中的方法的对比，本文方法好像要更好一些）![[Pasted image 20221216120652.png]]
## 定量实验
- 在某一方向上改变后对对应语义的变化进行评价，SeFa与superviesd方法表现差不多，但是更加effective和generalize![[Pasted image 20221216120445.png]]
# 启发
- 本文提出的方法，实际上是使用了另一种方式寻找Latent space上比较重要的direction，与前一篇文章具有比较强的相似之处。但不清楚本文引入Lagrange Multiplier之后运算出的结果与PCA运算的结果是否有某些数学上的联系。
- 思考：latent space上重要的方向是否就是图像某种semantic的低维embedding？