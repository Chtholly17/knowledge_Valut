# 概述
## 背景
- DISTS
- Dispersion index：离散系数，标准差（或方差）和均值之比
## 问题
- global方法缺少对自然图片中local structure 和 local texture的关注
## 贡献
- A-DISTS：使用dispersion index分割不同位置的structure和texture
# 具体工作
- Structure and Texture Separation.
	- Texture：在空间上是均匀的。structre：在空间上具有更精确的位置![[Pasted image 20230203154003.png]]
		- 对于小尺度的纹理，只需要一个很小的感受野就可以捕捉到其特征（比如上图中的a和b）。对于尺度较大的纹理，需要更大的感受野。
	- 对于VGG的每个stage，使用一个滑窗来计算局部的离散系数![[Pasted image 20230203154334.png]]
		- mu和sigma代表一个feature map上patch的标准差和均值，以上计算在spatial尺度上进行，多个channel取平均
		- 使用数据集测试，发现texture通常具有较小的离散系数，以下数据集中左边属于structure，右边属于texture![[Pasted image 20230203155604.png]]
			- 实验结果如下：深层的feature map中，离散系数较大的patch对应large scale的texture，浅层则对应small- scale的texture![[Pasted image 20230203155636.png]]
	- 对于每个patch，使用以下公式计算这个patch是texture 的可能性：![[Pasted image 20230203155337.png]]
		- w和b是fit on图片patch 数据集的参数
- Perceptual Distance Metric
	- SSIM![[Pasted image 20230203163910.png]]
	- $p^{(i)}$是这个patch属于texture的概率，取reference和input中的较小者，对应的q是属于structure的概率                                            ![[Pasted image 20230203163954.png]]
	- 分别计算结构与纹理相似度，并使用以上概率作为权重，对所有patch进行计算后求和![[Pasted image 20230203164301.png]]
	- 以上是一个scale中一个channel的结果，再对channel和scale逐个求和![[Pasted image 20230203164350.png]]
# 实验
## 定性实验
- 在图片上测试以上方法的效果，下图中，颜色越深，代表可能属于texture的可能性越大![[Pasted image 20230203155907.png]]
	- 采用浅层的feature计算，干草这种复杂的纹理会被当作结构特征，随着网络变深，干草这种复杂的纹理也别认为是纹理了
	- 当网络特别深，可能感受到的范围不仅仅包括干草，因此整张图片都偏蓝
- 将A-DISTS用作loss，进行SR，loss如下![[Pasted image 20230203164625.png]]
	- 结果![[Pasted image 20230203164642.png]]
- 与其他loss对比![[Pasted image 20230203164702.png]]
## 定量实验
- IQA性能测试![[Pasted image 20230203164440.png]]
- 2AFC测试![[Pasted image 20230203164516.png]]
# 启发
- 使用简单的数学方法区分texture和 structure，本质上是对DISTS更精确的优化。赋予了结构相似度和纹理相似度合适的权重
- 不太能直接用在我们目前的问题上。