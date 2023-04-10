# 概述
## 背景
- Perceptual Distance：描述两张图片的Perceptual Similarity，按照人类知觉定义
- 基于Perceptual Similarity思想的方法：SSIM、MMSIM、FSIM、HDR-VDP
## 问题
- 基于像素的similarity比较方式，忽略像素间相关性
- 构造一个perceptual metric非常困难
	- 人类对相似的判断依赖于高级结构信息
	- 与图片的内容有较强的相关性
	- 可能不能用距离衡量
## 贡献
- 建立了一个包含多种干扰图片的数据集（干扰方法包括几何方法与使用网络干扰），包含人类对于图片相似性的判断（different perceptual test and JND），使用这些数据，可以用校准预训练网络的feature输出
- 各种网络输出的feature map是对perceptual similarity很好的评价指标，与网络架构不相关
# 具体工作
- 构造了BAPPS Dataset
- 使用$l_2$ distance测量了不同网络对reference image和distort image之间的feature distance，对于reference patch $x_0$与distorted patch $x$，取计算每一层的特征图中（尺度为$R^{H*W*C}$），H与W相同对应位置长度为C的向量（$\hat y^l_{HW}$与$\hat y^l_{0HW}$分别代表distorted中的向量与reference中向量）的差值，其中$l$代表当前的层数。与另一个长度为C的向量$\omega _l$进行Hadamard乘法，并计算$l_2$ distance。
![[Pasted image 20221107202057.png]]![[Pasted image 20221107205036.png]]
- Learned Perceptual Image Patch Similarity(LPIPS)
	- 几种训练方法的变体
		- lin：保持预训练参数不变，train linear weights $\omega$，这一步在已有的feature space中进行"perceptual calibration"
		- tune：从一个预训练过的网络中初始化network，并允许网络的参数进行调整
		- scratch：随机初始化网络，完全使用本数据集训练
	- loss function：                                                                 ![[Pasted image 20221107212310.png]]
# 实验
- 数据集中的每个验证用例都是一个三元集合$(x,x_0,x_1)$
	- $x_0$和$x_1$分别表示使用两种不同方式干扰过后的图片，$x$代表原图
	- 需要从 $x_0$和$x_1$中选择认为更好的一张，结果可以是0或1
	- 每个三元集包含五个judgement，最后与本方法结果相同的judge占所有judge中的比例为这个评价方法所得到的分数。
## 定量结果
![[Pasted image 20221109165427.png]]
- Low-level 的方法（比如SSIM等），效果表现弱于所有的网络（无论结构、训练是否有监督），甚至弱于随机生成的网络，说明将网络结构与向信息更稠密方向的滤波器（可以理解为训练网络，使滤波器关注到更多深层perceptual信息，可以帮助网络进行知觉判断）
- 训练过的网络确实可以学习到人类知觉层面的信息。（在面对传统distortion时，更复杂的网络VGG的表现要优于简单的网络）
- 自监督的网络和有监督的网络效果表现差不多，说明不同的任务类型都需要相似的语义信息
- 使用传统distortion训练的网络可以迁移到对真实图像处理算法处理过后的图片的评价上，效果均好于low-lever的方法。但是使用一个高级任务（classification）等训练，并迁移（对参数进行tuning，就是作者提到的calibrating）到一个low-lever的任务上（比如2AFC等）却导致了效果的下降
![[Pasted image 20221109171523.png]]
![[Pasted image 20221109171532.png]]
- 2AFC与JND问题之间具有很强的相关性，说明2AFC的测试中也包含了一些人类perceptual层面的判断（尽管2AFC是low-lever的任务）
### 疑问
- 2AFC与classification和detection的关联系数和detection与classification之间的关联系数差不多，可以说明什么？
- 什么是detection和semantic task
## 定性结果
![[Pasted image 20221109174128.png]]
- BiGAN与SSIM对图像的评价，可以发现存在模糊时，GAN倾向于认为图片存在较远的差距，而SSIM认为二者相似（基于像素对齐）
### 疑问
- 为什么左上角第一张图片SSIM会认为相似度较高？存在明显的位移
# 启示
- 网络之间的信息感知具有一定的共性，朝向某种high-level任务训练的网络，都能捕捉到图像中共通的隐藏语义信息，好的feature具有迁移性
- low-level的任务，比如2AFC等，可能也涉及到了一些感知信息，但直接使用low-level task进行训练效果没有high-level好