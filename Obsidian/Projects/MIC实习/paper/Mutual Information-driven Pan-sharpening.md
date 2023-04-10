## 概述
- Pan-sharpening，看作一个fusion task，目前两种主要方法：image-level和feature-level![[Pasted image 20230320223151.png]]
## 问题
- 现有的方法没有显式地学习MS和PAN之间的互补信息，造成了信息的冗余（information redundancy），并且造成copy artifact的现象
## 贡献
- 提出一种互信息驱动的Pan- sharpening方法，首先将PAN和MS分别投射到一个modality-aware feature space，在这个空间中最小化它们之间的互信息
- MI，通过互信息最小化方法，减少MS和PAN的feature之间的冗余信息![[Pasted image 20230321151531.png]]
## 具体工作
- framework![[Pasted image 20230321143010.png]]
	- Embedding的方法：
		- 对MS和PAN卷出多个stage的特征![[Pasted image 20230321145902.png]]
		- 然后用一个3\*3卷积核卷一下
			- 第一层的feature，卷完之后直接FC![[Pasted image 20230321150043.png]]
			- 第n层的feature，卷完之后加上上一层FC之前卷的那一次的结果，再卷一次，再FC，各个卷积核之间的参数不共享![[Pasted image 20230321150049.png]]
	- Embedding之后计算mutual information，减少MS和PAN之间的信息冗余![[Pasted image 20230321150122.png]] ![[Pasted image 20230321150134.png]] ![[Pasted image 20230321150141.png]] ![[Pasted image 20230321150159.png]]
	- INN Block（invert neural network）：有效地对特征进行fusing（混合）
- Overall loss![[Pasted image 20230321150618.png]]
## 实验
- 定性实验![[Pasted image 20230321150632.png]]
- 定性实验![[Pasted image 20230321150731.png]]
- 消融实验：对INN block和MI作用的研究![[Pasted image 20230321151020.png]]
## 启发
- 存在冗余、重复信息的任务中，可以尝试使用本文中提出的方法，embedding + minimize MI
- 对于需要特征融合（fusing）的任务，可以考虑INN block，能提一些点![[Pasted image 20230321151118.png]]