## 背景
- 人脸生成算法
- SRM：一种滤波器，可以提取图像的high frequency noise
	- 图像的高频噪声，由于数字摄像头的性质产生，移除了图像的color content，并且描绘了图像的深层特征![[Pasted image 20230313192140.png]]
## 问题
- 鲁棒性不佳，采用与训练集不同的测试集，效果不够好![[Pasted image 20230313093019.png]]
	- CNN检测器过于关注图片的color texture信息，缺少对于高频信息的关注，不同算法生成的图片会具有不同的fake texture **（有研究发现CNN特征提取器明显偏向于texture）![[Pasted image 20230313191335.png]]
		- 以上是一些传统的两阶段方法
	- 使用grad-CAM maps，**提取图像中被CNN用于分类的区域**，跨域生成的情况下明显存在问题![[Pasted image 20230313093605.png]]
## 贡献
- 应用高频噪声进行生成人脸识别
- multi-scale high-frequency feature extraction module that extracts high-frequency noises at multiple scales and composes a novel modality.（多尺度高频noise特征提取器）
	- **将高通滤波器应用在低阶filter产生的feature上，对高频和低频成分分别处理
- residual-guided spatial attention module that guides the low-level RGB feature extractor to concentrate more on forgery traces from a new perspective（使低阶RGB feature 提取器更加关注伪造痕迹）
- crossmodality attention module that leverages the correlation between the two complementary modalities to promote feature learning for each other
## 具体工作
- framework![[Pasted image 20230313192627.png]]
	- 多尺度高频率特征采集的部分
		- 将原图像使用SRM核进行卷积，得到图像pixel domain的信号X_h
		- X_h送入卷积，产生两种Feature F和F_h，分别代表低频和高频的feature
		- 将F继续使用SRM进行卷积，然后加到F上，F继续进行下一层卷积
		- 最后的特征F_h包含了multi-scale的高频信号
## 实验
- 消融实验![[Pasted image 20230313194415.png]]