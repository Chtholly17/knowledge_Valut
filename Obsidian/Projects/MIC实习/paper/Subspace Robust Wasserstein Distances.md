# 概述
## 背景
- OT
- WD
	- Monge coupling: pi（可以理解为将x与y中的每个元素进行coupling）![[Pasted image 20221205195500.png]]
		- 从而可以按照如下的方式定义W-2距离，找最合适的使转移代价最小的coupling方案![[Pasted image 20221205201921.png]]
	- Transport plan，在两个不同distribution之间的转移函数![[Pasted image 20221205195855.png]]
		- 以Transport plan的角度对2-W距离重新定义：minimize trace(of second order displacement matrix) minimization，T(x)理解为将具有u分布的x transport到v分布上![[Pasted image 20221205195353.png]]
		- second order displacement matrix for coupling pi，这个矩阵可以代表在coupling pi下x与y的差异，W-2距离就是计算最小的差异![[Pasted image 20221205200526.png]]
- subspace
	- k-dimensional subspace  of Rd![[Pasted image 20221205200931.png]]
	- $P_E$：从$R_d$到E的正则投影
- Mahalanobis distance：马氏距离
## 问题
- a lack of robustness and instability of OT metrics with respect to their inputs.
## 贡献
- 提出一种OT问题的具有convex性质的近似解，将高维Distribution投射到k维空间，以计算转移矩阵（可以表示Transport paln）的前k大特征值之和的方式取代计算转移矩阵的trace（传统WD采用计算Trace的方式）
	- 可以认为，当投射到一维时（k=1），此方法等同于MAX-SWD
## 具体工作
- Subspace Robust Wasserstein Distances
	- PRW的定义：注意，PRW not convex![[Pasted image 20221205201210.png]]
		- 意思就是将uv投影到k维空间的面上，然后计算W-2距离
		- 这个定义很难计算
	- SRW的定义![[Pasted image 20221205201656.png]]
		- min-max problem，min是指transport cost最小，max指投影到的k维平面要让transport cost最大，实际上，不需要真正地投射到K维，而是在原本的维度上用一个$\Omega$矩阵进行约束，它的trace是k，也就是计算前K大的d维$V_{\pi}$的特征值的和
		- 关键是以下这个引理，SRW等价于求$V_{\pi}$前k大特征值的和![[Pasted image 20221205202937.png]]
		- 为了方便计算，使用Mahalanobis distance进行了以下定义，具体为什么和上面的式子等价并不清楚![[Pasted image 20221205220732.png]]
	- 二者的关系：SRW可以看做PRW的一种具有convex性质的宽松形式
- 一些相关的引理：
	- x，y的Dirac函数之间的SRW等于二者之间的l1距离![[Pasted image 20221205202841.png]]
	- ![[Pasted image 20221205211737.png]]
- 计算SRW的两种算法：
	- 超梯度方法：![[Pasted image 20221205211806.png]]
		- 优化一个concave function![[Pasted image 20221205212504.png]]
		- $\Omega$是一个projection（d\*d)
		- U：Vpi前k大的特征值对应的特征向量组成的矩阵（d\*k）
		- pi：从u到v（分别是x与y对应的distribution）的一个OT
		- Vpi：second order displacement matrix for coupling pi，W-2距离的目标就是优化它的迹
	- 使用Frank-Wolfe算法计算正则化的SRW![[Pasted image 20221205215206.png]]
- 总结：SWD就是投影到一维，并计算W-2，这时Vpi就是一个单独的数，它的迹也就是一个数，这就是两个patch之间的SWD。假如投射到k维并计算W-2，那就是PRW，SRW是PRW的一种方便计算的近似。近似方法是使用一个d维的矩阵，它的trace是k，用它对两个d维分布投影（还是投影到d维），然后解决一个min-max问题，目标是找到max Omega和min pi
- 并且SRW具有对称的性质，满足三角不等式
# 实验
- 随着采样点的增加，W-2距离和SRW距离差距减小![[Pasted image 20221205235946.png]]
- 点越多，误差越小![[Pasted image 20221206000051.png]]
- 不同space下的OT，左边是W-2，右边是SRW![[Pasted image 20221206000135.png]]
- SRW对噪声具有鲁棒性![[Pasted image 20221206000232.png]] ![[Pasted image 20221206000247.png]]
