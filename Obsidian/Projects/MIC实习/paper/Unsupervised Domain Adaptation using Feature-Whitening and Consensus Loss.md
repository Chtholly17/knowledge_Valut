# 概述
## 背景
- domain shift：predictors trained on a dataset do not perform well when applied to novel domains
- domain adaption methods（DA methods）：解决classifier因为domain shifter出现的性能下降问题
	- Unsupervised Domain Adaption methods（UDA）：通过对source和target的边缘分布进行对齐，来解决domain-shift问题
		- Correlation Alignment：使用一个loss计算source和target协方差矩阵的差距，协方差矩阵使用网络最后一层的输出计算
- BN-based layers：![[Pasted image 20230306192722.png]]
	- 假设一个minibatch $B={x_1,x_2,...,x_m}$，每个sample的维度为d，以上式子是对第i个sample的第k维的normalization
	- sigma和mu分别是这个minibatch中这一维的均值和标准差
	- gamma和beta是缩放和移动比例，是可学习参数
## 贡献
- Domain specific Whitening Transformation（DWT）：计算中间层feature的协方差矩阵，这些**中间层的作用是对数据进行白化（whitening），将feature投射到一个标准圆上。**
- Min-Entropy Consensus loss（MEC loss）：对同一个sample进行两种不同的扰动，计算这两种扰动版本的交叉熵或者平均欧氏距离
# 具体工作
- Framework![[Pasted image 20230306191946.png]]
- DWT：
	- 使用BW（Batch Whitening）替代BN（Batch Normalization）![[Pasted image 20230306211735.png]]
		- $W_B$白化矩阵，对batch中的每个sample去中心化之后白化，白化之后进行缩放和平移，和BN的区别在于使用白化后的feature替代了直接用feature
		- $\Omega=(\mu _B, \sum _B)$ ，$\sum _B$是使用B计算的协方差矩阵
	- 假设Bs和Bt分别是source和target domain 的一个batch输入网络之后，网络的任意一层的输出![[Pasted image 20230306212307.png]]
		- 这一步相当于使用Whitening Transformation将两个batch投射到了一个标准圆分布（common spherical distribution）
		- 使用简单的scale和shift代替了coloring操作，避免了额外的网络参数
		- **作者认为，对feature进行Whitening，能去除一个batch中数据的关联性，目的是使loss函数更加的平滑
- MEC loss：
	- 输入可以分为三个domain，分别为source domain（B^s），以及两种接受了不同方式扰动的target domain（$B_1^t$、$B_2^t$）
	- 其中$B_1^t$、$B_2^t$由于是从相同的target domain变化而来，因此在BWT阶段可以使用相同的$\Omega=(\mu _B, \sum _B)$ 
	- 对于Source Domain 的元素，直接计算cross-entropy loss，其中yi是对应sample对应的ground truth，直接以soft-max的形式计算预测结果等于ground truth的概率![[Pasted image 20230308220437.png]]
	- 对于两种perturbed target domain中的元素，应用如下的MEC loss![[Pasted image 20230308221149.png]]
		- 在所有label中，找到一个class z，其中包含的label使**两种不同扰动的target sample预测结果等于这类的概率之和最大
		- 在训练过程中，z会不断变化，MEC是一个动态变化的loss
		- 最终的loss为![[Pasted image 20230308222227.png]]
# 实验
- 定量实验
	- 与各种UDA方法对比![[Pasted image 20230308223434.png]]
# 启发
- MEC loss，由于两种pertubed target domain sample不会有太大的差别，比如轻微的几何位置变化也可以归于这种情况。
- 降低数据的相关性（decorrelation），可以让产生的loss更加平滑
