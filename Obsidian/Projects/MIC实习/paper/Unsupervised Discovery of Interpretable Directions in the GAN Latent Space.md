# 概述
## 背景
- direction in GAN latent space
## 问题
- 寻找interpretable direction需要surprised方法（人工提供label）或self-surprised方法
	- 需要人工标签或者pre-trained model
	- 只能找到研究者希望的方向，对于不同的方向可能需要重新训练
## 贡献
- completely unsurprised approach to discover interpretable direction in latent space.
	- model-agnostic 
	- does not require costly generator re-training.
- discovers new practically important directions, which would be difﬁcult to obtain with existing techniques.
# 具体工作
-  Motivation![[Pasted image 20230112203837.png]]
	- 在latent随机方向上移动导致出现多种互相影响的变化
	- 目标是找到某些direction，这些方向上的变化与其他方向正交，产生的图片与其他方向变化后产生的图片具有明显可区别的语义特征差别。
	- 核心思想：只会导致图像中一个因素变化的方向时interpretable 的
- Framework![[Pasted image 20230112204215.png]]
	- $e_k$，表示某一坐标轴的单位向量，长度为K，只有一个元素为1（第k个元素为1），其他都是0，代表某个坐标轴
	- $\epsilon$是一个标量，代表在这个坐标轴上移动的长度
	- 相当于：原始latent space向量z，沿着A的第K列对应的方向移动了距离$\epsilon$，得到latent space中的向量$z+A(\epsilon e_k)$
	- 与原始向量一起送入generator，得到两个输出图片，送入reconstructor网络R
		- R对于不同的数据集可以需要采用不同的model作为backbone
		- 两张输出图片在channe维度上连接后送入reconstructor，spatial size不变
		- 使用两个不同的head来分别输出$\hat{k}$和$\hat{\epsilon}$，$\hat{k}$是一个1到K之间的整数，$\hat{\epsilon}$是一个实数
		- 使用以下方程作为loss，一同优化A和R![[Pasted image 20230112213744.png]]
			- z、k、epsilon固定，求对A和R的均值，是一个二元函数，目的是求使loss最小的A和R
			- $L_{cl}$是交叉熵函数，$L_r$是MSE，lambda是可调节的超参数，Lr可以让沿着找到方向的变化具有连续性，避免以下这种将所有输入图片变化到类似结果的情况出现![[Pasted image 20230112222820.png]]
			- z从latent space中使用正态分布采样，k从1-K中均匀分布采样，epsilon从-6-6范围的实数中均匀采样，每次迭代中k、epsilon重新采样
	- 相当于一次从K个方向中，使用R进行评价，找到这K个方向中与其他方向可区分度最大的方向。
# 实验
- 分别使用了MNIST数据集+Spectral Norm GAN Generator、AnimeFaces数据集+Spectral Norm GAN Generator、CelebA-HQ数据集+ProgGan generator、ILSVRC数据集+BigGangenerator
- 使用Adam优化器
- Evaluation metrics
	- 由于难以直接评价方向的可解释性和独立性，本文提出了以下两种测试方法
	- Reconstructor Classiﬁcation Accuracy (RCA)，更高的RCA意味着找到的方向更容易和其他方向区分，为了得到RCA，将A设置成随机矩阵并且在训练过程中不优化A
	- Individual interpretability (mean-opinion-score, MOS），对于某一方向h，人工评价以下两点
		- 对于不同的z（随机初始化），是否h方向上的移动具有相同效果
		- h是否导致单一factor的变化，可以解释吗？
		如果以上两点都成立，将方向标记为1，否则标记为0.
## 定性结果
- 训练过程对方向的优化：逐渐值关注某一个可区分、解释性高的方向![[Pasted image 20230112224736.png]]
- 在MINST+Spectral Norm Gan上的实验结果：![[Pasted image 20230112224756.png]]
- AnimeFaces+Spectral Norm Gan上的实验结果：![[Pasted image 20230112224822.png]]
- CelebA+ProgGan的实验结果：![[Pasted image 20230112224936.png]]
- BigGan上的实验结果：![[Pasted image 20230112225000.png]]
## 定量结果
- 在BigGAN上让实验人员将找到的方向导致的变化分为三类，比例如下![[Pasted image 20230112225215.png]]
	- Random方向很难找到Textual方向上的变化
	- 本文提出的方法对于各种方面的变化方向都可以找到
- 对不同方式获取的方向的RCA和MOS比较![[Pasted image 20230112224348.png]]
	- Random：随机方向
	- Coordinate：latent space坐标轴方向
## 消融实验
- K的值的研究
	- K的值过小，会导致reconstructor过于容易训练和分类，导致得到的方向并不独立（可能涉及到多个factor的变化），更大的K可能会导致找到更多的方向，但不会影响可解释性（因为每个找到的方向都比较独立）![[Pasted image 20230112230116.png]]
- 对Loss function中lambda的研究：
	- lambda更大，就更关注epsilon预测的准确程度，而epsilon预测得不准确，实际上就是在这个方向上移动不同的距离可能导致出现所谓不连续的变化，这可能会导致将不同图片移动后输出同一结果的情况。大的lambda就是为了避免找到的direction出现这种情况。
	- 如果lambda过大，则会导致对方向这种性质的关注程度过高，忽略了对k准确预测的关注，而k预测的准确性强调了这个方向的独立性，也就是这个方向的变化更容易和其他方向区分，所以lambda过大也会导致效果不好。![[Pasted image 20230112230934.png]]
# 启发
- 可以作为一种寻找GAN latent space中direction的方法，应用在我们的系统中，找到的direction可以看作在latent space中对某种语义信息的编码。