# 概述
## 背景
- Image restoration：需要一个loss function用于测量被损坏后的图片和参考图片之间的distance
## 问题
- 采用现有的distance作为loss function效果不佳
	- $L_2$距离进行梯度下降导致生成的图像模糊，缺少细节
- GAN：难以训练，对于不同图像可能需要重复训练
- WD：随着维数增高，需要投影的次数指数级别增长(如何优化projection的方向)，实际上只有一些投影方向可以很好地反映我们需要的信息，大量重复并平均只是为了尽可能逼近这个方向。
- Perceputal Loss：弱化成一个mean问题，由于很多存在high- resolution的图片会导致相同的low- resolution observation结果，因此对low- resolution的优化可能变成求这些high- resolution的平均，导致模糊![[Pasted image 20221124213724.png]]
- Contextual Loss：我们已经学习过，计算各个patch之间的Cosine Distance，找出最相近者并求和，被证明类似于KL散度，对distribution不够敏感
## 贡献
- 提出了一种基于distribution distance 的loss function（PDL），无需空间上的对齐（同一张低解析度的图片，可能有许多distribution基本相似但是空间位置存在细小差异的高解析度图片与之对应，使用L2距离会产生这些众多高解析度图片的平均结果，因此结果可能比较模糊。
# 具体工作
- PDL：Projection Distribution loss![[Pasted image 20221124173422.png]]
	- 首先计算VGG feature
	- 进行投影
	- 首先加上两张图片之间计算出的Lq距离（实现中q=1），与接下来的结果相加![[IMG_2499.jpg]]
- 公式：![[Pasted image 20221124210124.png]]
## 疑问：
- 计算perceptual部分的时候，是采用哪一个scale的feature进行的计算？还是所有的scale都计算了求和？
# 实验
## 定性实验
- Toy example：当对分布产生位移时，JS散度和KL散度保持不变，WD增加，说明WD确实更加关注distribution上的差异![[Pasted image 20221124170556.png]]
- 高低分辨率的图片，使用卷积产生的各层feature map，KL散度结果基本不变，而且比较类似。WD距离，在低维下（通常认为是对几何形状等低维信息比较敏感的层级）比较小，而高维下（可能涉及到高维的语意信息等）分布差距较大。说明WD更关注distribution的shape而不仅仅是低维上几何形状的shape![[Pasted image 20221124170943.png]]
- 使用PDL以梯度下降的方式进行降噪，发现增大L1 Loss的权重并不能提升降噪的效果![[Pasted image 20221125010739.png]]
- 采用各种Loss进行降噪，调整perceptual 的权重系数，发现虽然no- perceptual具有更高的SSIM和PSNR，但是看起来更加模糊![[Pasted image 20221125142501.png]]
## 定量实验
- 训练速度：和使用L1距离作为Loss的VGG网络差不多![[Pasted image 20221125010151.png]]
- 在校验集上的评分，采用多种方式计算feature map之间的distance（perceptual），no- perceptual意味着$\lambda$ 等于0.![[Pasted image 20221125010807.png]]
	- no-perceputal拥有更高的PSNR以及SSIM，但是看起来很模糊，在perceptual的评价方式中采用不同权重系数的PDL表现最佳
	- contextual也可以看作一种近似衡量Distribution Distance的方法
- 人类对于两种不同方法进行降噪后的图片进行评分，给出人认为更好的图片的百分比，PDL与其他各种方法相比表现更好![[Pasted image 20221125142827.png]]

# 启发
- 不同于SWD，PDL采用了多维度融合的方式进行投影，相较于其他类似于Contextual Loss以及L1 Loss计算perceptual loss的方法，确实有了一些提升。但是PLD还是没有解决的问题在于：还是需要进行多次投影，以蒙特卡洛方式试图找到最佳的投影方向。并且在比较中也没有与传统的SWD计算出的Loss进行比较。
- 我们还需要考虑的是：
	- 这一种跨维度计算边缘分布的方法，相较于SWD各个channel维度单独投影，是否能感知到更综合的信息，产生更好的效果？如果好的话，能否考虑将线性投影换成类似于GSW中的非线性投影？
	- SWD单个channel维度单独投影，是否有更好的确定patch的方法
	- 不管是SWD还是PDL，能否对投影方向进行优化，找到最合适的投影方向，抛弃蒙特卡洛方法？