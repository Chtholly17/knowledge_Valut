# 概述
## 背景
- Stereo matching：立体匹配算法，也称为双目深度估计，输出的视差图可以用于深度图的计算
	- matching cost computation：匹配代价计算，计算匹配像素和候选像素之间的相关性，与视差和深度的计算有关（计算disparity map）
	- cost volume：可能是一种计算匹配代价的方法，使用图像的特征计算时，需要尽可能保证两张具有一定视差的图片的feature尽可能类似
## 问题：
- 当前的匹配代价计算方法在生成深度图像上的效果不好，关键是对于没见过的图像（unseen domain）计算方法的鲁棒性欠佳
## 贡献
- Feature Consistency Stereo networks（FCStereo）
	- Challenges：
		- obtaining a high feature consistency on the training set
			- Stereo Contrastive Feature（SCF）loss：约束生成的feature，使对应的点在feature空间中的位置尽可能接近
		- generalizing this consistency across different domains![[Pasted image 20230302204423.png]]
			- Stereo Selective Whitening（SSW）loss：抑制会随着视角变化而发生改变的信息 **（可以近似理解为结构信息？随着观察角度的变化而发生较为明显的改变，而纹理信息则不会这样）
# 具体工作
- Framework：![[Pasted image 20230302210827.png]]
	- SCF：
		- Positive Pair：left和right image中映射到feature上同一点的两个低维向量，从训练集提供的ground truth disparity d（视差图）中获得
		- Negative Pair：在一个区域内随机采样
		- Pixel-wise contrastive loss![[Pasted image 20230302211255.png]]
			- $F(u,v)$代表采样获得的negative set
			- $\phi ^{l}_{u,v}$代表feature上的一个点在低维对应的一个向量
			- $\phi ^{r}_{u-d,v}$代表 $\phi ^{l}_{u,v}$的postive pair
		- Non-matching region removal![[Pasted image 20230302212344.png]]
		- 求和![[Pasted image 20230302212353.png]]
	- SSW：
		- 使用Instrance Normalization（IN）代替Batch Normalization（BN），对于每个sample分别进行normalization，对于每个sample![[Pasted image 20230302213003.png]]，进行如下的normalization![[Pasted image 20230302213018.png]]
			- 方差和均值分别是数据集中所有sample在channel i上的均值和方差 ![[Pasted image 20230302213552.png]]
		- 对于每个sample，计算variance matrix，就是标准化之后的feature和自己的转置矩阵相乘，**这个矩阵实际上![[Pasted image 20230302213626.png]]
		- 对于送入的一批采样，首先先对left和right的invariance matrix平均，然后按如下方式运算并求和![[Pasted image 20230302215648.png]]
			- 最后得到一批sample的invariance matrix，其中$V_{i,j}$的数值表示，该层feature的i、j两个channel之间的相关性对视角改变（可以理解为导致非对齐的几何变化，而语义信息没改变）的敏感性
				- 也就是这个数值越大，在产生一定视角改变时，i、j两个channel之间的相关性会发生很大的改变
		- 计算一个mask矩阵，只保留V矩阵中具有最高的invariance value的数值![[Pasted image 20230302220306.png]]
		- 最后计算Lssw![[Pasted image 20230302220320.png]]
			- $\hat M$ 是一个上三角矩阵，因为mask矩阵是对称的，因此只采用上半部分就够了
			- $\gamma$是layer的标志，代表对对应层输出的feature进行以上的计算，Lssw需要对各个层的计算结果求和
			- ![[Pasted image 20230302220744.png]]实际上衡量了feature间各个channel之间的相关性？使用以上loss进行训练，目标是使各个feature的各个channel之间的相关性尽可能地高
# 实验
- 定性实验![[Pasted image 20230302221448.png]]
- 定量实验
	- 对feature consistency的实验![[Pasted image 20230302222109.png]]
- 消融实验：在不同框架下使用本文提出的方法![[Pasted image 20230302221755.png]]
# 启发
- 我们可以考虑将left和right图片当作我们任务中移动过的图片和原图，然后使用本文提出的方法，这样训练可以使发生位移的图片和原图的feature具有较小的差异。那使用这种方法训练好的网络是否可以看作一个编码器，可以用于提取图像中那些不会随着位移而产生明显变化的特征（比如texture）？