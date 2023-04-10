## 背景
- 对于非配对数据的Super Resolution，应用在相干断层扫描血管造影(OCTA)图像的处理上
## 问题
- 超分方法容易丢失low-resolution图像的高频信息（相关文章Fritsche, M., Gu, S., Timofte, R.: Frequency separation for real-world superresolution）
- 利用频域信息的超分辨率方法，相关文章有：
	- Kim, G., et al.: Unsupervised real-world super resolution with cycle generative adversarial network and domain discriminator.
	- Unsupervised real-world image super resolution via domain-distance aware training.
- 现有对OCTA的处理方法难以处理非配对的问题
## 贡献
- 基于非配对的问题，使用频域和sptial domain的信息进行超分辨率工作
- inverse consistency：使用一个gan生成HR，另一个GAN使用HR生成LR
## 具体工作
- framework![[Pasted image 20230312210741.png]]
	- 灰色的shallow是upsample和downsample module，分别用于上采样和下采样
	- 使用Fourier变换，之后分解出高频低频的信息（推测可能是FFT、mask、IFFT）
		- 低频直接使用
		- 高频进行HFB操作，相当于平均高频成分和原图片![[Pasted image 20230312212735.png]]
	- 低频使用浅层网络、高频用深层网络提取feature，低频高频分别对待的原因可能是为了加强对LR低频的重视
		- 浅层网络只有三层卷积层
		- 深层网络具有八个残差blocks
	- 将低频、高频的feature连接到一起
	- 连接之后的feature进行upsample，得到HR，HR进行相同操作，区别在于对高频用浅的网络提取特征，而对低频用深的网络提取特征，然后连接到一起，downsample
- 评价器的结构![[Pasted image 20230312213522.png]]
	- DWT（discrete wavelet transformation）产生LL、LH、HL、HH
	- HR评价高频成分，LR只评价低频成分。对应之前说GAN进行SR工作产生的HR会丢失LR的低频信息
- Identity Loss：HR产生的LR和LR产生的LR之间的L1![[Pasted image 20230312214221.png]]
## 实验
- 与state of art对比![[Pasted image 20230312214318.png]]
## 启发
- 图像的高频与低频信息的区别对待已经非常广泛，我们的工作重点是要说明对特征的高低频进行分解有什么含义。是否可以从感受野的角度，说特征的高频低频成分，实际上反应了图像更大尺度上的分布性质？**
- 在感受野较大的feature中进行分解的方式，进一步消除了非对齐的问题