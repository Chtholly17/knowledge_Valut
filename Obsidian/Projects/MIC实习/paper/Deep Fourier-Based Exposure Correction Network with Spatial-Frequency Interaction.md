## 背景
- Exposure Correction：曝光矫正，错误曝光的图片往往具有光线和结构信息的损失
- Motivation：
	- 图像进行FFT分解之后的结果，光照信息一般包含在幅值图（Amplitude）中
	- phase中一般包含structure或者semantic information，受光照的影响较少![[Pasted image 20230321165435.png]]
## 问题
- 大多数现有的Exposure Correction方法都在spatial domain上进行
## 贡献
- Fourier-based Exposure Correction Network（FECNet）：一种sptaial- frequency interaction的方法，用于曝光矫正
	- 包含amplitude sub-network和Phase sub- network，分别用于重建光照和结果信息
	- Spatial- Frequency Interaction（SFI）block：用local spatial information和global frequency information相互融合
## 具体工作
- framework![[Pasted image 20230321170227.png]]
	- Amplitude Sub-Network：encode-decode结构，Xnormal为ground truth，输出为Xout1
		- 使用Xout1计算L_{s1}，要求![[Pasted image 20230321181655.png]]
			- X_{out}的光照信息（Amplitude）和GT尽可能相近
			- 使用GT的lighting和X_{in}的structure（Phase）进行IFFT，产生的结果应该尽量和X_{out1}相近
		- 整个Amplitude Subnetwork以幅值信息为导向，目标是训练一个良好的encode-decode结构，使其输出保留原图的phase和GT的Amplitude
	- 使用X_{in}的phase和x_{out1}的Amp进行IFFT，送入Phase sub-network，理由是为了避免引入去他的Phase信息（avoiding introduce the altered phase component）
		- Phase sub-network：以Phase主导，进行1\*1的卷积，输出为X_{out2}
		- 计算Ls2要求输出和GT尽可能相似，并且和GT拥有尽可能接近的Phase![[Pasted image 20230321182240.png]]
	- Overall loss                                                 ![[Pasted image 20230321182438.png]]
- Spatial-Frequency Interaction Block（SFI）![[Pasted image 20230321183731.png]]
	- SFI Block：结合了频域的全局信息和空域的局部信息
	- 先使用输入feature的Amplitude，经过1x1卷积，然后和自己的相位IFFT![[Pasted image 20230321191307.png]]
	- W1和W2代表3x3卷积![[Pasted image 20230321191639.png]]
	- 对SFI模块中的各个feature可视化，可以发现各个成分分别保留和包含了各种不同的信息![[Pasted image 20230321191847.png]]
## 实验
- 定性实验![[Pasted image 20230321193242.png]]
- 定量实验![[Pasted image 20230321193337.png]]
## 启发
- 频域和空域的结合、频域中相位和幅值的处理
	- 还是抓住Local和Global的描述，注意如何很好地解释频域和空域信息的结合
	- 相位和幅值分别代表了某种信息，在某个阶段保留某种信息