# 概述
## 背景
- OT问题（optimal Transport）：求两个概率分布之间相互转换的最小代价，可以体现两个distribution之间的相似性
- WD、SWD：SWD是投影+排序之后的一维WD计算
## 问题
- traditional OT-based method（例如WD）：计算复杂性很高
- SWD：对WD的优化与近似，但随着维数的增加，为了保证获取到平滑的结构特征，需要的随机线性投影数量快速增加，带来了非常大的计算复杂性
## 贡献
- 扩展了sliced-Wasserstein distance，证明使用polynomial projection同样可以产生有效的distribution metric
- 确认了几种GSW是一个distance function的情况（对projection 的要求）
- 提出MAX-GSW distance，使用一个投影给出投影空间中的最大距离，极大减少了GSW计算的运算量
# 具体工作
- GWD与MAX-GWD的计算：
	- GSW：首先将XY分割成多个patch，每个patch计算投影后的结果，randon transform 的参数为$\theta _l$，使用g投影后进行升序排序，计算WD，重复L次结果取平均
	- MAX-GSW：使用梯度下降求出一个$\theta$，使distributio之间的距离最大，求出这个最大的距离即可。
![[Pasted image 20221115222605.png]]
![[Pasted image 20221115222625.png]]
- g：Generalized Radon Transform（GRT），Radon Transform将高维的分布投影到一维平面中，可以用如下的方式定义：![[Pasted image 20221115222758.png]]
	- $\theta$ 函数g的参数，$g(\theta)$可以表示一个高维超平面，g的函数形式需要提前确定，比如SWD中是线性函数，其中$$g(\theta)= A_1x_1 + A_2x_2 + A_3x_3 + ...$$$$\theta = {A_1,A_2,A_3,...}$$$\theta$从高斯分布中随机采样，g函数的形式提前确定，polynomial degree3代表的就是最高次数为3的多项式。
	- 高维分布在这个超平面上的投影是一维的：![[Pasted image 20221115222924.png]]
	- 在满足以下四个条件时，通过g定义的projection可以表示两个分布之间的距离（GWD此时是一个表示距离的函数），如果需要使用GSW与MAX-GSW，需要保证投影g拥有以下的性质![[Pasted image 20221115223043.png]]
## 疑问
- 是否高维的超平面（上图中的第一种Projection）就对应了Linear Projection？
	- 是的SWD对应一个高维的超平面，因此是线性projection
# 实验
## 定量实验
- 使用GSW梯度下降的方式，寻找$$min_{\mu }GSW_p(\mu , \nu)$$，其中$\mu$采用正态分布初始化，$\nu$是target distribution，本实验分别对$\nu$是多种分布的情形进行了测试![[Pasted image 20221116011546.png]]
	- 同时，针对GSW，分别采用了四种define function（之前提到的g，可以用来表示一个投影），分别是Linear（此时等价于SWD）、Circular、Polynomial degree3与Polynomial degree 5![[Pasted image 20221116012341.png]]
	- 纵轴是相似程度（使用2-Wasserstein distance表示）的对数，越小表示相似程度越高
	- 可以发现采用Polynomial总体而言效果较好
- 使用SWAE架构对于GSW与其他Wasserstein Distance进行了对比
	- 属于VAE的一种，使encoder的满足某种分布
	- SWAE：Sliced-Wasserstein Distance Auto-Encoder，对输入信号进行编码（其实就是一个projection，一般采用CNN实现），使encode之后的data符合一种特定的分布（用z表示特定的分布），也就是求解以下方程：![[Pasted image 20221116013029.png]]
	 - $p_x$表示对x进行sample，变成patch的集合
	 - 原图与DEcoder输出，Encoder结果和z分布尽可能相同（使用KL散度进行计算）
	 - $\gamma _{\phi}$与$\gamma _{\Phi}$分别对应encoder与decoder的参数，$\phi$与$\Phi$对应两次投影映射，$\Phi(x,\gamma _{\Phi})$与$\phi(\Phi(x,\gamma _{\Phi}),\gamma_ \phi)$分别对应使用encoder对x进行projection产生的信号（理论上应当符合特定的分布）以及对产生的信号使用decoder再次进行projection生成的信号（理论上应该与原本的x尽可能接近），使用GSW代替这里的SW![[Pasted image 20221116013651.png]]
	- 在本次实验中，规定z为如上图所示的Swiss Roll分布，进行测试结果如下：![[Pasted image 20221116013907.png]]
	- 同样使用2-Wasserstein distance，可以发现，使用Polynomial的GSWAE效果最好，分布情况最符合z
	- 对Swiss Roll与Ring（环形）分布进行了测试，得到的结果如下![[Pasted image 20221116014134.png]]
	- 可以发现，在decoder输出与x的分布上，MAX-GSWAE（使用MAX-GSW代替SW）表现最好，在encoder输出和z的分布上，WGAN的效果最好，但MAX-GSWAE效果也相当不错。
# 启发
- 对g的优化，显然，采用不同的define function对distribution similarity的评价效果不同，某种程度上MAX-GSW反应了人最能关注到的差异。但本次实验中采用的是W2距离作为评判标准，其本身不一定符合HVS的特点，是否可以找到更合适的define function（g），使其在视觉表现上具有更好的效果呢？
- W2距离本身可以优化，可不可以将其变成一个迭代过程，将用这种方法优化的距离进一步送入此系统中，用求MAX的方法多次迭代？这个过程**如何约束**，使其更加符合人类的认知？
- 更进一步，g是否存在最优解，g可以理解为一个高维的平面，高维的分布投射在上面的同时，各类信息的保留程度不尽相同（从上面对同一分布投射产生不同的一维分布也可以看出），是否g使投射的过程中尽可能保留人眼所更关注的信息？