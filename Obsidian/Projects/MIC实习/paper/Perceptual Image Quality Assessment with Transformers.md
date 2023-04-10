# 概述
## 背景
* Transformer是一种基于self-attention的算法，最先应用在NLP领域，用于文本翻译工作。
* 现有的IQA算法（SSIM、PSNR）等大多使用视觉层面（图像像素的RGB数值）来判断图像的质量。
* Vision Transformer采用CNN提取的特征图像替代传统Transformer的直接信号输入，这一架构同样可以应用在IQA领域。
* FR
## 问题
* 传统IQA算法（PSNR、SSIM）难以预测由GAN生成的图片质量
* 图像感知层面的质量（Perceptual quality）包含高级的特征信息，使用传统算法难以评估。
* 目前缺少将Transformer应用在IQA问题上的尝试
## 贡献
* 设计了基于Transformer的IQA算法，使用CNN网络提取图像深度特征信息进行embedding，并且研究了输入标准信号、干扰信号和差值信号（$f_{ref}$、$f_{dist}$、$f_{diff}$）对预测结果的影响

# 方法模型
* framework
![[Pasted image 20221102102824.png]]
使用CNN提取目标图像特征，特征信号用二维卷积层进行投影，分割成$H*W*256$的256通道特征图，然后将其按像素对齐并裁切，得到的向量进行embedding之后送入Transformer Encoder和Decoder，以上的Encoder、Decoder、MLP Head是标准的Transformer架构。
* 具体模块
	* CNN特征提取网络：
		* Extract Feature Map with same shape$R^{H*W*C}$，$C=6*320$
		* Use both reference image and distort image，also use the difference signal$$
		f_{diff}=f_{ref}-f_{dist}
		$$
	* Embedding：
		* 对于输入给Transformer Encoder和Decoder的信号，使用了不同的特征embedding方法，额外进行了quality embedding和trainable position embedding
	* Classic Transformer Architecture
# 实验
## 定性结果
* 输出attention map的可视化结果，attention map由所有encoder与decoder中的attention weights平均求得
![[Pasted image 20221102105720.png]]
* 使用PIPAL数据集上的一组reference image和系列distort image，将distort image按照MOS评分递减排列，使用多种算法估计评分并比较效果![[Pasted image 20221102110110.png]]
## 定量结果
* 使用KADDID-10k数据集训练IQT，在LIVE、CSIQ、TID2013、PIPAL数据集中对IQT进行了测试，其中PIPAL数据集中包含大量由GAN生成的图片
![[Pasted image 20221102110407.png]]
* 使用SRCC与KRCC指标，对多种IQA方法在LIVE、CSIQ、TID2013、PIPAL数据集上进行了测试并进行对比
![[Pasted image 20221102110556.png]]
![[Pasted image 20221102110609.png]]
## 消融实验
* 测试了输入$f_{ref}$、$f_{dist}$、$f_{diff}$对预测结果的影响，发现当输入中包含$f_{diff}$时效果较好![[Pasted image 20221102111037.png]]
* 测试了输入图片视觉层面的信号（RGB值）和特征层面的信号（texture map）的效果，发现对于diff信号而言，输入特征层面的diff信号更加有用![[Pasted image 20221102111455.png]]
# 启示
* Transformer作为最先应用在NLP领域的算法模型，同样也可以应用在IQA这样与图像相关的领域，可能的原因在于，无论是处理自然语言还是图片，都需要感知其中的语义信息，这正是Transformer模型所擅长的。而相对于自然语言，图像中的语义信息隐含在图像结构中，往常的方法（比如SSIM）尝试直接从视觉层面去提取这样的语义信息。而使用CNN可以提取图像的特征信息，相较于视觉层面，这其中可能包含了更多描述图像本质特征的语义。因此，对于涉及到感知层面的IQA问题，首先需要考虑的就是图像深层语义信息的提取与分析，本文给我们提供的思路值得参考。