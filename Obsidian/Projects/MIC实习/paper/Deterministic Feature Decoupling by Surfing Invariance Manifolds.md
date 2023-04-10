# 概述
## 背景
- 非解耦的feature：在数值上与其他feature有关联
- 输入向量的某种结构不一定只由某种特定的feature所控制，可能是受多种feature 的影响
## 贡献
- 一种特征解耦方法，使用一种normalize将global feature进行一系列的分解
# 具体工作
- 定义
	-  Invariance manifold：从N维度原始数据空间采样并排序，得到一个向量x0，将x0沿着global feature 的梯度方向移动，在原始数据空间能得到的全部向量的集合。Invarinance manifold可以用以下式子表示![[Pasted image 20230207222016.png]]
	- Goal manifold：特征由多个维度为M的向量组成，假设$v^{des}$是包含我们想要的值的向量，经过映射能得到$v^{des}$的所有原始N维数据空间向量的集合。![[Pasted image 20230207221715.png]]  ![[Pasted image 20230207221747.png]]  
	- Feature translation：将x0沿着invariance manifold移动，直到找到和desired goal manifold的交集中距离x0最近的一个向量（从x0出发，在invariance上移动的距离尽可能短）。这个过程叫做Feature translation。![[Pasted image 20230207221959.png]]
		- 其中![[Pasted image 20230207222222.png]]代表一个manifold distance，代表invariance manifold中两个点之间的最短距离
	- Reference manifold 和 Normalization：假设![[Pasted image 20230208145514.png]]对于所有N维的向量x都存在。那么$v^{ref}$就是reference manifold，以上这个特殊的translation就是normalization（normalization是以reference manifold为goal 为goal manifold使用的特殊translation），使用以下公式定义reference maniflod![[Pasted image 20230207223340.png]]
- 通过normalization来Decoupling global features
	- 下图中，x1与x2进行normalize后得到相同的向量（normalize实际上就是找投影后得到v1ref的maniflod之间的交点）。两个映射函数f1和f2，分别对x1（或者Invariance manifold上的其他任何向量）和x1正则化的结果进行投射，在这两个向量处这两个函数的梯度相互垂直。说明在此处，两个不同投影产生的特征之间是正交的。![[Pasted image 20230208154539.png]]
- Decoupling为什么可以提升可解释性
	- 每对于个投影函数，输入一个N维向量，我们都希望其有一个正确的输出（假设为向量c）。文中证明了以下公式成立，也就是将输入normalization之后，输出和目标c之间的差距这个向量，对于不同的投影函数而言是相互正交的。![[Pasted image 20230208160335.png]]
	- 下图是一个这样的例子![[Pasted image 20230208160437.png]]
- 总结：本文方法的核心思路是使用normalization实现decoupling，对于所有的输入向量先进行normalization，得到的feature可认为是相互正交，数值上不互相依赖的
# 实验
- 横轴为采样数量，纵轴为MSE，对比各种估计dataset distribution方法的性能![[Pasted image 20230208162950.png]]
# 启发
- 可以尝试将本特征解耦方法应用在目前的工作中，对输出的特征进行分解。需要看看这篇文章代码的具体实现。