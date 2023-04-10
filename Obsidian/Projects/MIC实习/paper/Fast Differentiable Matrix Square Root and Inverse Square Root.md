# 概述
## 背景
- pricinple square root of Matrix(and inverse square root)![[Pasted image 20230303102752.png]]
	- pricinple square root：可以用于减少特征在主成分方向上的变化（使feature在主成分方向上的分布更加密集）
		- 通常，使用SVD计算pricinple square root![[Pasted image 20230303103334.png]]
	- inverse square root：可以用于对数据进行whiten，使数据在每个维度上均匀变化
## 问题
- 现有的SVD分解方法，应用在深度学习任务中存在一些问题，以下式子是求loss相对于feature 的梯度，公式中的K矩阵存在$K_{i,j}=\frac{1}{\lambda _i - \lambda _j}$，当两个奇异值过于接近时，可能会产生梯度爆炸![[Pasted image 20230303103717.png]]
- 现有的训练框架进行奇异值分解的速度太慢，需要消耗更多的时间
## 贡献
- 提出了一种快速计算principal square root和inverse square root的方法
# 具体工作
- Fast  Principal Square Root：
	- Taylor serirs in Matrix Form![[Pasted image 20230303154159.png]]
	- 使用泰勒级数表示出$A^{\frac{1}{2}}$ ![[Pasted image 20230303154256.png]]
	- 
