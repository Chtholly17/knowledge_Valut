# 概述
## 背景
- set- structured data：a set of desperate points（point cloud）or features extracted from a network，目标在于模型的输出结果不受set中元素的排列顺序的影响
- implicit method：需要定义一个集合之间测量distance或者similarity的方法
	- 将input与output set之间的元素一一对应（如何确定这种对应关系，解决一个相关问题）
	- 计算每个pair之间的distance或者similartiy
- explicit method：建立一种与set间元素排列顺序无关的mapping，将set中的元素投射到hilbert space，提供一个固定维度的structured set的表达方式（可以理解为建立一个函数，输入一个set，得到一个固定维度的向量），这个函数应当具有可微、连续的性质，以便于使用在ML问题中
	- hilbert space：[希尔伯特空间_百度百科](https://baike.baidu.com/item/%E5%B8%8C%E5%B0%94%E4%BC%AF%E7%89%B9%E7%A9%BA%E9%97%B4/5630049)
## 问题
- structured data embedding需要面对的challenge：需要与顺序无关
## 贡献
- 推出PSWE（Pooling by Sliced-Wasserstein Embedding）,使用SWD进行Pooling，在backbone网络较为复杂时，简单的pooling（例如Mean pooling）效果很好，但当backbone相对简单时，PSWE表现明显更好
- 相当于对WD（近似）进行了euclidean embedding
# 具体工作
- frame work![[Pasted image 20221125171249.png]]
	- input：forward过程中，某一层输出的feature map产生的distribution以及density
	- ref：参数回传后，网络希望在这一层产生的feature的distribution（使最后输出的结果可能更加接近label，实际上最后一层的输出也可以看作具有distribution，label表示成向量也应该具有distribution，应该使这两个分布尽量接近）
- Algorithm![[Pasted image 20221125173216.png]]
	- 每一层，进行采样并投影，进行projection到一个structure set，计算两个set之间的SWD，每两个patch之间的SWD作为Pooling对应点的结果
	- 需要注意的是：投影（每一层进行一次投影），所有投影向量（L层一共有L个）参数是可以训练的，具体怎么训练不是很明白
# 实验
- 没有仔细看，网络的复杂程度和Pooling的复杂程度二者有一个比较复杂即可，都太复杂了容易过拟合
# 启发
- SWD的投影方向，无需每次进行随机采样投影向量，可以使用这种方式，在一次初始化之后可以进行训练，使投影的方向拟合某一个特定的方向，这个方向可以拟合L1距离，也可以拟合人类的感知（使用IQA数据集）