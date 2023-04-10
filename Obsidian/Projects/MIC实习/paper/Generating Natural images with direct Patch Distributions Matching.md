# 概述
## 背景
- 图像生成问题
- SWD：Sliced Wasserstein Distance，计算图像之间distribution的相似程度
## 问题
- GPNN：基于Local Patch的方法，无论是单向还是双向similarity（BDS），无法比较图像patch distribution的相似程度，可能生成具有相同patch但distribution具有较大差别的图片。并且BDS计算复杂，具有$O(n^2)$的时间复杂度
- GAN：Generative Adversarial Network，存在mode collapse问题，生成的图像可能只包含很少的几种patch，并且GAN运算复杂，对于不同的任务需要重复训练
## 贡献
- 基于SWD，设计了将图像中的patch使用随机卷积投射为高维空间的向量并计算SWD的方法，可以用于衡量图像之间distribution的差距
- 使用类似于GPNN和SINGAN的结构，使用SWD梯度下降代替nearest  neighbor或者GAN，在保持效果的前提下大幅提升了运算速度
# 具体工作
## Framework
![[Pasted image 20221111162128.png]]
![[Pasted image 20221111161427.png]]
- 迭代过程
	- $\tilde y_N$ 是Input图像加上随机blur等处理后的结果
	- 对input多次down scale，将最底层结果与 $\tilde y_N$ 共同作为输入（其中$x_N$作为target），进行SWD梯度下降
	- 结果送入上一层作为input，多次迭代后生成最终的输出图像
- 具体模块
	- SWD的计算                                                                    ![[Pasted image 20221111162155.png]]  ![[Pasted image 20221111162331.png]]
		- $P^{\omega}$是图像P使用随机投影（一个没有训练的网络投影到高维空间对应的向量），$S\tilde WD(P,Q)$采用多个随机网络进行投影并计算均值（相当于计算数学期望）
	- 算法框架![[Pasted image 20221111213036.png]]
		- $\omega$ 是保持一定分布的随机向量，生成K次计算均值（表示一种数学期望）
		- 每个patch投影到高位空间中的点，拉平成一维向量，进行排序，更新Loss
		- k次循环结束后，进行梯度下降
# 实验结果
## 定性结果
- GPNN与GPDM效果好于sinGan，GPDM速度更快![[Pasted image 20221111163436.png]]
- ![[Pasted image 20221111163656.png]]
- ![[Pasted image 20221111163723.png]]
- ![[Pasted image 20221111164342.png]]
	- 采用了近似最近的GPNN，提升了速度但损失了对patch的相似程度，并且distributuin信息保留效果差（眼部结构不清晰），同时近似的GPNN无法使用GPU进行加速，操作难以并行。GPDM解决了此问题，且运算速度具有非常大的提升
## 定量结果
![[Pasted image 20221111165319.png]]
- Diversity：计算SFID的参数
- SFID：评估生成图片的分数，越低越好
- NIQE：评估生成图片的分数，越高越好
# 启发
- SWD：用于计算图片distribution的差距，同样也可以应用在FR IQA问题中
- 采用网络可以将特征投射到高维，通常对空间变化不敏感，并且卷积每一层的输出都可能包含了不同的feature信息，组合计算可以获取更完整的图片特征
# 疑问
- flipped version of the filter
- how to sort patches
	- 每个patch会被投影到高维空间中的一个点，直接依靠点的数值进行排序
![[Pasted image 20221111171839.png]]