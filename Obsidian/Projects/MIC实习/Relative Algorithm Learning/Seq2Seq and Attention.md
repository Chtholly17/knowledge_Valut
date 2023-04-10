[Seq2Seq模型和Attention机制 - machine-learning-notes (gitbook.io)](https://luweikxy.gitbook.io/machine-learning-notes/seq2seq-and-attention-mechanism)
# Seq2Seq模型
![[Pasted image 20221105215426.png]]
- 三部分组成：**Encoder、Decoder 和连接两者的 State Vector (中间状态向量) C 。**
- 类似于RNN的many-to-many结构，区别在于输出和输入可以不等长，实现了**从一个序列到另外一个序列的转换**，输入与输出的长度可变，因此适合用在翻译问题上
- 该结构是最简单的结构，**Decoder 的第一个时刻只用到了 Encoder 最后输出的中间状态变量**
![[Pasted image 20221105220501.png]]
- 相对复杂的结构，将Context当作Decoder每一时刻的输入：
![[Pasted image 20221105220509.png]]
# Encoder 和 Decoder
- 在现在的深度学习领域当中，通常的做法是将输入的源Sequence编码到一个中间的context当中（Encoder），这个context是一个特定长度的编码（可以理解为一个向量），然后再通过这个context还原成一个输出的目标Sequence（Decoder）。![[Pasted image 20221105220111.png]]
- Encoder-Decoder结构的局限性：
1. Encoder和Decoder的唯一联系只有语义编码C
	- 中间语义向量无法完全表达整个输入序列的信息。
	-   随着输入信息长度的增加，由于Context向量长度固定，先前编码好的信息会被后来的信息覆盖，丢失很多信息。
2. 不同位置的单词贡献相同
	- Decoder过程，其输出的产生如下：$$\begin{aligned} &y_1=g(C,h^{'}_0)\\ &y_2=g(C,y_1)\\ &y_3=g(C,y_1,y_2) \end{aligned}$$
		明显可以发现在生成y1、y2、y3时，语义编码C对它们所产生的贡献都是一样的，实际上在翻译某一个单词时，输入信息中应该有部分信息的影响大于其他部分。
# Attention机制原理
## Attention的作用
- Attention模型：Decoder不再将整个输入序列编码为固定长度的中间语义向量C，**而是根据当前生成的新单词计算新的Ci，使得每个时刻输入不同的C**。
![[Pasted image 20221105221449.png]]
- $a_{ij}$Encoder中隐藏层$h_j$与第$i$阶段的相关性（第i阶段可以理解为翻译第i个单词），比如当输入3个单词时，Encoder按这样的方式生成每次解码需要的$C_i$![[Pasted image 20221105221926.png]]
## 如何得到Attention
* Encoder中第j个隐向量$h_j$与Decoder中第i-1个隐向量$h^{'}_{i-1}$计算得到，比如$a_{1j}$的计算过程如下：![[Pasted image 20221105222317.png]]
* 推导过程：
	- 语义向量$c_i$由Encoder的隐向量加权求和得到：                               ![[Pasted image 20221105222738.png]]
	- 每个隐向量$h_j$的权重$\alpha _{ij}$按照以下公式计算                              ![[Pasted image 20221105222837.png]]
		- $T_x$为Encoder中的隐向量数量
		- $e_{ij}=\alpha (s_{i-1},h_j)$，其中$s_{i-1}$就是$h^{'}_{i-1}$
	- $\alpha$函数每个论文中不同，可以用网络计算，也可以用二次型矩阵计算，训练时主要就是训练这个用来计算$\alpha$的网络                           ![[Pasted image 20221105223326.png]] ![[Pasted image 20221105223405.png]]
 - Attention效果如下图所示（attention map）![[Pasted image 20221105223453.png]]
