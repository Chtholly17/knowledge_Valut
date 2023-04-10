# 概述
## 背景
- GAN
	- BigGAN：
		- 图片I具有可能的分布p(z)，z看作I映射到latent space的向量
		- 第一层网络接受z作为输入，生成$y_1 = G(z)$作为feature map中的一个vector
		- 逐层网络接受上一层的输出，bigGAN会同时将初始的vector z作为输入（类似于ResNet）![[Pasted image 20221216001425.png]]
		- bigGAN同样可接受一系列的vector作为输入
	- styleGAN：
		- 第一层接受一个常向量$y_0$作为输入，每一层加上一个z的非线性转化作为输入![[Pasted image 20221216001801.png]]
		- 让每一层具有不同的omega，可以产生style mix
		- 其中，M是一个多层感知机                                                                                                       ![[Pasted image 20221216002817.png]]      
- Principal Component Analysis（PCA）
	- 将高维数据降低到K维，方法与SRW类似，算法如下：![[Pasted image 20221212113648.png]]https://zhuanlan.zhihu.com/p/77151308
## 问题
- training GAN with labeled images, and add user control over the output focus on supervised learning of latent directions. 需要高昂的人力成本对GAN进行监督
## 贡献
- 低层级的GAN输出的feature tensor的Principal Component可能代表了z space（采样自一张图片）中某些重要的方向，可能代表了图片的某些重要特征
# 具体工作
## PCA在GAN space上的计算
- style-GAN：
	- 对图片高维分布采样N次，产生N个z向量
	- 每个z向量使用M进行计算，产生N个omega特征向量，对它们进行主成分分析得到矩阵V
	- 给出一个以omega定义的图片（这个图片采样z后计算出的向量为omega），将其送入生成网络之前可以进行如下的变换：                                                                              ![[Pasted image 20221216003302.png]]
		- x：PCA coordinates，x矩阵的entry（矩阵中的每个元素）都是一个控制参数，可以由用户自行定义（初始时是0）
- BigGAN：
	- 更为复杂，因为没有学习到z的分布，就没有一个可以表示图片的向量omega，在中间层进行PCA计算
	- 对于每一层的输出同样采样N次，N个z向量产生N个y向量（Activation space）![[Pasted image 20221216004124.png]]
	- 使用N个feature tensor进行PCA，产生矩阵V（是Activation space上的主方向），同时计算出数据均值mu（应该也是一个N维向量，可能是每个y取平均），每个feature tensor y经过以下投影得到它的PCA坐标，注意xj（yj的PCA coordinates也是一个矩阵）![[Pasted image 20221216004635.png]]
	- 采用线性回归的方式将其转化回latent space，具体公式如下：![[Pasted image 20221216005610.png]]
		- $x_j^k$是$z_j$对应的PCA coordinates的第k个坐标
	- 整个矩阵U的计算方式如下：![[Pasted image 20221216005900.png]]
	- 这样U的每一列uk与feature space PCA矩阵的一列vk对应，可以称uk是latent space的一个主方向（principal direction）
	- 以上过程的可视化结果如下：![[Pasted image 20221216010254.png]]
	- 采用类似于style GAN的方式对图像进行编辑，不同于styleGAN，此编辑在每一层的feature上独立进行                             ![[Pasted image 20221216010605.png]]
- 实验中的发现：
	- 只有前20大的主方向上的改变可能导致大规模的几何变化，类似于纹理、背景颜色等变化通常需要修改更靠后的principal direction
	- StyleGANv2的latent distribution p(w)具有近似的简单结构，principal coordinates呈现近似独立的非高斯单峰分布
	- 前100的principal direction足以描述所有图像的某种特征方向
	- bigGAN的主成分组成与图片类别有关，各个类别之间是独立的，同一类图片的组成成分可能非常接近。对不同类别的图片进行相同的全局改变可能导致完全不同的结果。
	- 主成分可能与GAN采用的training set有关，使用FFHQ face dataset训练的styleGAN2，在前3大主方向上改变可能导致旋转。而没有找到可以产生位移的主方向，可能是因为training set进行了严格的对齐
	- 许多不同方向的修改可能具有内在联系，比如修改狗的朝向可能导致它张嘴
	- GAN可能“不允许”一些改变，比如在成年人脸上添加皱纹会增加他的年龄，而在孩子脸上却不会造成这种效果
# 实验
## 定性实验
- 以下三行表示了图片的三个最主要的principal direction，红框标出的是对应方向上的原图。可以发现这三个方向上的调整分别会对性别、旋转角度等发生改变![[Pasted image 20221216013915.png]]
- 分别只在有限的GAN输出层对主方向进行调整，第一行代表只将对主方向进行的调整应用在0-2层上![[Pasted image 20221216014323.png]]
	- 可以观察到只对头的转向产生了影响（第二行只对发色产生了影响）说明不同layer输出的feature的相同主方向可能具有不同的semantic信息
- 在某些方向进行修改的进一步实验![[Pasted image 20221216015242.png]]
- styleGAN与BigGAN的对比实验![[Pasted image 20221216015923.png]]
## 疑问
- 我们目前还没办法确定具体是什么信息，这其中可以总结出什么普遍的规律吗？比如是否颜色在深层feature中会被看作是相对而言不重要的方向，而几何变动在低层网络中会被看作比较重要的方向
# 启发
- GAN空间中不同principal direction可以看作是对图像的某种特征的一种embedding，我们可以采用这样的方法对图像的高阶特征进行提取，但对这些高级的特征进行怎样的比较适合用作衡量图像之间差异的标准呢（是简单的计算L1距离还是使用其他距离计算方法）？但可能只在同一类图片中这种方法才有意义（class independent）
- 可能可以将GAN space中提取出的feature应用到我们的任务中。
- latent space上各个向量的采集是否采用蒙特卡洛方法，是否有更好的优化策略？