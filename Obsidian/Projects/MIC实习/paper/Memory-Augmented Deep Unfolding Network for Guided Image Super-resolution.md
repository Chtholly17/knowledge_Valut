## 背景
- Guided Image Super- Resolution（GISR）：obtain a high-resolution target image by enhancing the spatial resolution of a low-resolution target image under the guidance of a HR image.（和PAN- Sharpening有点像）
	- 有两个HR图片：HR guidance和HR target（类似于PAN和HRMS）
	- HR guidance：structural details can help enhance the spatial resolution of the LR target image
- 在GISR中，通常假设LR是target HR经过某个blurring kernel卷积之后进行downsample，再加上某个高斯模糊后的结果，其中ns是一个方差为sigma的高斯分布![[Pasted image 20230321162731.png]]
	- 可以表示成以下这种形式![[Pasted image 20230321162800.png]]
	- 研究LR的分布，如以下公式所示：![[Pasted image 20230321162922.png]]
## 问题
- 现有的方法将整个图像看作整体，忽略了guidance HR和target HR之间的非局部特征
## 贡献
- 提出了一种新的GISR方法
	- 对两种HR 使用MAP（Maximum Posteriori Estimation），使用两种先验信息
		- local implicit prior
		- global implicit prior
	- 提出了一个新的framework
	- 利用LSTM(Long short-term memory unit)减少stage之间的information loss
## 具体工作
- framework![[Pasted image 20230321162310.png]]
- 两种隐式先验：由于Guidance HR和target HR基本捕捉的是同一个场景，因此它们之间存在的相近的pattern
- 后续先跳过，太复杂，和当前任务相关性不大