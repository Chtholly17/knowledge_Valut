[Self-Attention和Transformer - machine-learning-notes (gitbook.io)](https://luweikxy.gitbook.io/machine-learning-notes/self-attention-and-transformer)
# RNN Encoder-Decoder模型
![[Pasted image 20221101210152.png]]
* 梯度消失问题
* 计算具有时间顺序
	* 时间片t的计算依赖t-1的结果，模型缺乏并行性
	* 顺序计算丢失信息，对于特别长期的依赖现象无能为力
* Transformer对应的解决方案：
	* 使用了**Attention机制**，将序列中的任意两个位置之间的距离缩小为一个常量
	* 不是类似RNN的顺序结构，因此具有**更好的并行性，符合现有的GPU框架**
# Transformer模型架构
* Transformer中抛弃了传统的CNN和RNN，整个网络结构完全是由Attention机制组成。
![[Pasted image 20221101210614.png]]
