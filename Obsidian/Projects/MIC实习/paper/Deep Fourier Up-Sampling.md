## 背景
- Up- Sampling：在卷积网络中被广泛用于多尺度建模
- spatial domain：local similarity and cross-scale position invariant properties，空域上的up- sampling运用了相邻像素之间的关系
## 问题
- 在spatial domain上的up-sample算子（interpolation、transposed convolution、un-pooling）严重依赖于局部像素attention
- Fourier domain上的up-sampling，不包含spatial domain上的局部特性，因此更具有挑战性（直接对频域进行down sample、up sample（使用spatial domain上的基于相邻像素关系方法），再进行逆傅立叶变换，可能无法得到正确结果）
## 贡献
- motivation![[Pasted image 20230315205319.png]]
- Deep Fourier Up-sampling：generic operator that can be directly integrated with the existing networks in a plug-and-play manner
- 本文提出的上采样方法可以直接应用在现有框架中：existing networks could achieve consistent performance improvement across multiple computer vision tasks
## 具体工作
- Definition：
	- f、g分别是图片和它在空间上进行2倍上采样（2-times zero-inserted up-sample）的结果（f长、宽是g的二倍）
	- F、G分别是f、g进行DFT的结果
	- H：对G在频域上进行二倍area-interpolation up-sample的结果
	- h：傅立叶逆变化的结果
-  Theorem-1：F（Up-sample之后的频域图）可以看作G每个点数值扩大四倍并重复四份的结果![[Pasted image 20230316102634.png]]
- Theorem-2：![[Pasted image 20230316103047.png]]
- Theorem-3：![[Pasted image 20230316103905.png]]
- 实现：![[Pasted image 20230316104219.png]]
## 实验结果
- 定性实验：![[Pasted image 20230316113034.png]]
## 总结和启发
- 提出了一种在频域上进行上采样的方法，后续在网络中可考虑用于替换传统的空间维度上采样。
- 在图像恢复类型的任务上效果不错
- 是否可以使用本文提出的三条theorem解释我们的实验结果？