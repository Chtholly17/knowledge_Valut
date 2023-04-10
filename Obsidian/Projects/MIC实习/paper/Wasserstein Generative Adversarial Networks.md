# 概述
## 背景
- unsupervised learning：自动提取目标的分布（distribution），随机生成distribution并进行迭代。当前的分布与目标分布尽可能的逼近。关键在于如何评价分布之间的距离
- Distribution Distance：
	- KL散度
	- JS散度
	- WD...
## 问题
- Kullback-Leibler divergence（KL散度）：非对称，目标分布的密度可能不存在，并且对偏移不敏感，本质上是求两个分布之间的相对熵。![[Pasted image 20221118212737.png]]
- Jensen-Shannon (JS) divergence（JR散度）：解决了KL散度不对称的问题，但KL的其他缺点（可能不连续、密度可能不存在等）依然存在![[Pasted image 20221118212930.png]]
- VAE：也需要加入随机噪声
- GAN：训练时计算开销大，不稳定，必须平衡评价器和生成器的性能。并且由于JS散度并不能保证loss-function连续可微，因此无法提供有意义的梯度。训练效果较差
## 贡献
- WGAN，使用W-1距离取代传统GAN中的JS 散度。
# 具体工作
- 评价器的训练目标![[Pasted image 20221118214459.png]]
- 生成器的训练目标                                                     ![[Pasted image 20221118214950.png]]
- WGAN算法![[Pasted image 20221118214605.png]]
	- $n_{critic}$训练多次Generator，只训练一次Discriminator，确保生成器与评价器的学习速度相匹配
	- 采样，输入网络的是一个向量。x是从目标数据的分布中随机采样得到的向量。z是从高斯分布中随机采样得到的向量（高斯分布初始化）
	- clip：限制backforward过程中参数最大的更新范围，防止初始时过大的梯度使参数更新太快，导致模型无法收敛。
	- 评价器梯度下降方向为：使目标采样向量的评分与求当前采样向量的评分差距尽可能地大（相当于尽可能为生成图片打低分），$f_\omega$表示评价器网络
	- 生成器的训练方向：使生成的图像尽可能可以在评价器中取得更高的分数，$g_\theta$表示生成器网络
# 实验
## 定性实验
- 学习给定的分布（8-Gaussian）![[Pasted image 20221118182741.png]]
	- GAN：由于JR散度可能存在不连续的现象，无法持续对评价器与生成器进行梯度下降，在一定Epoch之后无法再学习到更多的Distribution特征（Mode Collapse）
	- Unrolled GAN，另一种GAN，文中没有详细介绍，效果强于传统GAN，但效果弱于WGAN
	- WGAN，优先学习低维的Distribution特征，呈现出一个渐进的圆，之后再学习Density的分布，出现颜色的深浅
- 对Generator和Discriminator的robust![[Pasted image 20221118183640.png]]
	- 采用相同的Discriminator结构
		- 采用DCGAN的生成器，WGAN与GAN都具有较好的效果
		- 采用DCGAN 但是去除了batch normalization 和 保持恒定的filter数目（Generator的能力极大降低），GAN表现很差，WGAN基本不受影响
		- 采用MLP Generator，WGAN保持效果，效果略低于DCGAN生成器。而GAN出现了严重的mode Collapse，效果较差
## 定量实验
- GAN与WGAN评价器的梯度变化：GAN提供的梯度下降过快，并且由于LOSS可能不连续或不可微，导致梯度没有实用价值![[Pasted image 20221118184444.png]]
- 测试WD-1距离和JS散度的实际意义，发现WD-1更具有实际应用价值，符合人眼的视觉感受![[Pasted image 20221118220140.png]]  ![[Pasted image 20221118220343.png]]
	- 分别使用MLP、DCGAN、MLP（learning rate过大）Generator进行训练，中间不断输出生成的图片以及生成图片和目标分布之间的Distance，发现WD减少的时候，图片质量确实更高
	- JS散度不能提供有意义的距离，图片的质量变化，JS散度值基本不变
# 启示
- 对GAN原理的具体学习和各种Distribution Distance的学习


