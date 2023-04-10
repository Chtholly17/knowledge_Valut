## 背景
- Pan-sharpening：遥感图像融合
	- Pan picture：全摄图
	- LRMS：低空间分辨率多光谱图（LRMS）
	- 由PAN和LRMS融合得到高空间分辨率多光谱图（HRMS）
- Motivation：
	- PAN图和Ground truth之间的DFT Amplitude图相减![[Pasted image 20230319215949.png]]
		- PAN的Phase和GT的Phase更像，**因为PAN图和LRMS相比拥有更多的空间细节信息，而傅立叶变化产生的Phase图像描绘了原图的structure information**
		- 从最后一列可以看出，PAN图和Ground Truth之间的差别主要是在低频信息上（集中在中间），而LRMS和GT的区别在高低频都有，因此作者认为MS中缺少的高频信息可以从PAN图中获取
- **启发：我们的motivation是否也可以采用这样的方法，比较unaligned-data之间的DFT图像？
## 问题
- 大多数的Pan- sharpening只在spatial domain上进行，忽略了频域的信息
- Pan- sharping是PAN-guided LRMS image SR问题，**SR问题与频域高度相关，因为在下采样的过程中丢失了高频信息
## 贡献
- Spatial-Frequency Information Integration Network（SFIIN）：
	- spatial-domain information branch：直接在spatial domain使用卷积综合PAN和LRMS图像的局部信息
	- frequency-domain information branch：采用deep fourier transformation，利用全局contextual information
	- Dual domain information interaction：综合频域和空域的信息
## 具体工作
- framework![[Pasted image 20230319221725.png]]
- Spatial and Frequency Integration Block![[Pasted image 20230320202757.png]]
	- Frequency Domain Information Branch
		- 对于输入的feature，首先进行傅立叶变化，分解出幅值和相位成分![[Pasted image 20230320203824.png]]
		- 然后，将MS和PAN的幅值、相位分别Concat到一起![[Pasted image 20230320203908.png]]
		- 进行多次1\*1卷积，之后进行逆傅立叶变化
	- Spatial Domain Information Branch：将MS和PAN的feature CONCATE到一起，3\*3的卷积，**学习到局部的特征$F_{spa}$**
	- Dual Domain Information Interaction：从compensate和integrate的角度讲故事
		- Information Compensation
			- 从spatial domain和spectral domain提取出可区分的信息，用spatial局部信息补充spectral全局信息
			- 首先将频域和空域信息相减，利用Spatial Attention Mechanism（SA）to exploit the inter-spatial dependencies，输出spatial attention map（代表在spatial维度上收到关注的位置）![[Pasted image 20230320212001.png]]
		- Information Integration
			- CA：Channel Attention（exploit the inter-channel relationship，providing the more informative feature representation）
			- 最后又加上MS本身的featufeature（residual learning mechanism）![[Pasted image 20230320212254.png]]
- Loss Function：频域和空域的L1 loss
	- spatial domain                                                                           ![[Pasted image 20230320212537.png]]
	- frequency domain![[Pasted image 20230320212610.png]]
	- overral：拥有神秘的参数lambda，set to 0.1 empirically          ![[Pasted image 20230320212629.png]]
## 实验
- 生成图片IQA![[Pasted image 20230320212816.png]]
- 定性实验（最后一行是GT和生成图片之间的MSE）![[Pasted image 20230320213023.png]]
- 消融实验：Core Building Module的数量对实验结果的影响![[Pasted image 20230320213727.png]]
	- 说明深层的频域（global）信息还是有作用的
- 频域和时域feature的可视化![[Pasted image 20230320214045.png]]
	- 频域和空域feature分别展示了global与local信息，而Ffuse具有更丰富、综合的信息
## 启发
- 我们的motivation：
	- 首先使用spectral domain的信息，可以解释为使用global contextual information，global information对空间不对齐的问题比较适用
	- 然后做一些实验，说明只适用spectral信息是不足的，可能产生一些什么问题（缺少local信息）
	- 最后我们的方法从global和local information的角度出发