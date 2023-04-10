# Learning in the Frequency Domain
## Metadata
* Item Type: [[Article]] 
* Authors: [[Kai Xu]], [[Minghai Qin]], [[Fei Sun]], [[Yuhao Wang]], [[Yen-Kuang Chen]], [[Fengbo Ren]] 
 
* Date Added: [[2023-03-25]] 
#* Tags: #cv, #fourier-domain, #unread 
## Zotero Links
* PDF Attachments
	- [Xu et al. - Learning in the Frequency Domain.pdf](zotero://open-pdf/library/items/7DR82SSJ) 
* [Local library](zotero://select/items/1_V6NR3PBB) 
## Inroduction
### Background
- CNN
- DCT
- SE Block：[SE block 由一系列 SE block 组成，一个 SE block 的过程分为 Squeeze（压缩）和 Excitation（激发）两个步骤。其中 Squeeze 通过在 Feature Map 层上执行 Global Average Pooling 得到当前 Feature Map 的全局压缩特征向量，Excitation 通过两层全连接得到 Feature Map 中每个通道的权值，并将加权后的 Feature Map 作为下一层网络的输入，也称为 SE 通道注意力机制](https://zhuanlan.zhihu.com/p/358791335)
### Drawbacks
- 大多数CNN，需要将图片downsample 到224 \* 224作为输入，这个过程中丢失了一些信息，导致性能的下降
### Contribution
- 提出了一种在频域学习的方法，将图片进行DCT之后送进CNN
- 从频域的角度分析，发现CNN对DCT分解之后的low- frequency channel更加敏感
- 提出了一种channel selection方法，移除不必要的频域成分输入，在减少运算量的同时基本保持了性能
## Porposed Method
- framework![[Pasted image 20230327170921.png]]
	- 图像转变到YCbCr空间并进行DCT
	- 进行reshape，All components of the same frequency are grouped into one channel.
	- 进行channel selection，只有一些channel **（此时不同channel代表了不同的频率成分）** 中包含了比较重要的频率成分（在HVS中更容易被察觉）
- Channel Selection![[Pasted image 20230327162000.png]]
	- employ a dynamic gate module that assigns a binary score to each frequency channel(突出的channel被标记为1，其他通道被标记为0)
	- two-layer squeeze-and-excitation block（SE Block）：从Tensor1到Tensor3的变化类似于一个2层SE Block
		- 首先对所有通道（Tensor1）进行avg-pooling，得到Tensor2
		- 然后对Tensor2进行1\*1的卷积，得到Tensor3
	- 有两个可学习参数，分别用它乘以Tensor 3中的每个元素，得到Tensor 4（两个Tensor），对Tensor 4进行Normalize，分别代表每个通道可能会被标记为0或者1的概率
		- （两个不同的参数，乘上Tensor3，分别代表标记为0或者1的概率）
	- 然后用Bernoulli distribution Bern(p)（类似于2项分布），来决定特定的channel被标记为0还是1，p是由上一步中的两个可学习参数计算出来的
		- 但Bernoulli sampling不可微，因此使用Gumbel Softmax trick替代，实现了梯度回传
- Overall loss：![[Pasted image 20230327165033.png]]
	- $F(x_i) \in \{0,1\}$ ，代表第i个通道是否被选中
	- $L_{Acc}$是和预测准确度有关的loss
	- loss的第二部分用来训练SE Block，这样相当于鼓励SE Block选择尽量少的channel
- Static Channel Selection：
	- 以下是作者在几个分类任务上训练网络之后，绘制的channel selection heatmap，格子中的数字越大，表示代表的频率成分的频率越高![[Pasted image 20230327165622.png]]
	- 基本上网络选择的都是低频信息，说明低频信息在这些任务中更加重要
	- 基于以上实验结果，本文提出后续的频域学习方法可以只保留低频成分，减少送进CNN的数据量，但与视觉相关的信息量没有太大损失
## Experimental
- 定量结果：Image Classification![[Pasted image 20230327170021.png]]
- 定性结果：instance segmentation![[Pasted image 20230327170418.png]]
## Conclusions
- 将DCT方法，使用每个channel表示一个频域成分，然后用SE  Attention机制，得到每个channel的权重，这样就得到了每个频域成分的权值
- Dense Classification任务中，low- frequency信息可能更加重要
- 方法非常巧妙，我们可以学到的有：
	- 使用DCT，使每个channel代表一种频率成分
	- SE Block是运用在Channel之间的Attention机制
	- 在Channel Selection的部分中，本文主打从减少计算量的角度出发是合理的，**但这个方法也可以作为每个channel的权重，在deep fourier compoments中可能有用
- 这个能不能和我们目前的方案结合一下？
	- 我们能不能用SE Block机制，High-Light某些频域成分？但是SE Block是需要对激活层的参数进行学习的，恐怕比较难以应用在我们的metric中
	- **根据本文的结果，可能低频成分在这些任务中更加重要，我们可以尝试在metric中将低频成分的权重调大一些