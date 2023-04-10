## 背景
- 使用深度特征作为损失函数（perceptual loss）
## 问题
- perceptual loss具有以下问题：
	- 计算复杂
	- Domain Specificity：在一个domain上训练的特征提取器，在其他d
	- omain中提取出的语义信息不一定有效
	- Interpret Ability：不可解释性，选择哪一层的参数进行计算难以确定
## 贡献
- 证明perceptual loss等价于两张图片的small patch所具有的local distribution之间的maximum mean discrepancy（MMD）
- 基于以上证明，提出了一个方法：直接使用MMD计算两张图片的之间的距离，可以获得和perceptual loss相近的结果
## 具体工作
- Perceptual loss（使用随机参数的S-CNN）趋近于两个image的small patch之间的MMD
	- MMD的定义：f是一个评价函数（具体可以用一个VGG实现），其再生核为K，一般情况下K使用RBF kernel，**此时$||f||_H$越大，频域的不同频域成分之间的幅值差异越大，相当于评价l了图像的smoothness**![[Pasted image 20230324214142.png]]

	- MMD的近似![[Pasted image 20230324220659.png]]
	- **随机参数的infinite S-CNN，使用其作为feature extractor提取特征，计算池化层feature之间的l2距离，相当于计算图片之间的MMD**
## 实验
- mean image generation：生成和左边一系列所有图片之间距离之和最小的图片，采用不同的loss，可以发现L2导致模糊，而perceptual loss（使用不同的网络作为特征提取器）和本文提出的方法（MMD）都产生了较好的结果![[Pasted image 20230324164443.png]]
- 定量：SR![[Pasted image 20230324221753.png]]
- 讨论：mean image，左侧的图片具有明显不同的不同时，MMD效果很差，文中对此的解释为：
	- **MMD近似于采用VGG最后一层feature的perceptual loss，拥有的信息还是太少，当distribution之间差距较大时，优化MMD可能无法起到语义上平均的效果**
	- 当输入的分布差距过大时，训练过（就算是只有一层参数是训练过的）的VGG和random VGG，计算的loss会产生很大的差别
## 启发
- 某种意义上：**随机的VGG提取的feature之间的距离反应了图片之间局部（分成很多个patch）之间的distribution之间的距离**
- 局部分布的SWD？感觉和我们当前的解释比较难结合
- 咱们的metric中，可能可以用随机网络？毕竟输入的图片之间应该差别不大
	- 是不是可以用本文中的观点：随机网络perceptual loss等价于局部分布之间的距离，那么当pixel  domain上的局部分布之间差距不大的时候，我们的操作是否high-light了某种信息（某种semantic 、style、空间不敏感的信息）？