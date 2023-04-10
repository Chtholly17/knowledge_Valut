# 概述
## 背景
- image-to-image translation problem
- cycle- consistency：在cycleGan中提出，在image-to-image translation问题中被用作loss![[Pasted image 20230202194111.png]]
	- F：X domain to Y domain，G：Y domain to X domain
- InfoNCE Loss：
	- 将patch与对应的patch对应，并且将这个patch与其他patch disassociate
-  mutual information：![[Pasted image 20230202200905.png]]
- Contrastive Learning：![[Pasted image 20230202201806.png]]
## 问题
- cycle- consistency：假设两个域之间的关系是双射的，过于严格。当一个domain的图片和另一个domain的图片相比具有更多的信息时，这样的双射关系难以建立。
## 贡献
- 提出了通过最大化输入和输出图片的patch之间的互信息的方法，应用在contrastive learning中，可以替代cycle- consistency
## 具体工作
- 输入图像进行encode后接着decode，生成函数G由encoder与decoder两部分组成![[Pasted image 20230202202126.png]]
- framework![[Pasted image 20230202202326.png]]
	- 输出图像y中，采集一个patch
	- 输入图像x，同一位置采集一个positive patch，以及在其他N个位置采集的negative patch
	- 使用G网络中的的encoder部分，以及一个两层的MLP，将上述patch投射到embedding空间中。输出的feature stack中，spatial中每个位置对应的向量代表原图中的一个patch（更深的网络，这样的一个向量代表更大的patch）
	- 转化为一个N+1类分类问题，计算交叉熵loss，代表positive patch在N+1个patch中被选中的概率（互信息最大化）![[Pasted image 20230202203243.png]]
- Loss的计算：
	- PatchNCE loss：![[Pasted image 20230202210502.png]]
		- Generator 的Encoder网络具有L层，l层对应输出的feature stack spatial尺度上具有$S_l$个location
		- $\hat{z^s_l}$是output image在l层空间位置为s处的一个向量，H是MLP![[Pasted image 20230202211955.png]]
		- $z^s_l$为input image相同层、相同spatial location处对应的向量，$z^{S/s}_l$为input image在l层面其他的所有向量，作为negative
		- 本文中也提到了另一种loss，使用external patch（其他图像中采集的patch作为negative）。这种loss 的表现效果不如上述的internal loss。
	- 模型最终的loss：![[Pasted image 20230203000611.png]]
		- $L_{GAN}$：adversarial loss![[Pasted image 20230203001947.png]]
- Final objective：![[Pasted image 20230202211735.png]]
	- 采用以上domain Y的PatchNCE loss，防止generator做出不必要的改变（可能是对除了纹理之外的物体结构等发生改变）
# 实验
## 定性实验
- image-to-image实验，本文提出的CUT与FAST CUT方法与其他方法对比![[Pasted image 20230203125454.png]]
	- 失败的例子：
		- 第一行：没有识别出主体，将texture应用在了背景上
		- 第二行：网络创造了个舌头
	- 失败的分析：训练集中可能存在舌头图片，对输入图片的简单匹配可能导致这种情况出现
- Genc学习的可视化：都较好地关注到了主体的部分![[Pasted image 20230203152343.png]]
# 启发
-  相对于使用SWD会导致patch之间空间信息的丢失，采用PatchNCE loss使patch对齐，是否能在一定程度上限制GAN的随机生成现象，更多地保留原图中的语义。
- 同时，像PatchNCE loss这样逐个patch对应的计算方法是否可以用作一种IQA的手段？既允许了一定的不对齐，也可以保证整体语义信息不发生太大的改变。
