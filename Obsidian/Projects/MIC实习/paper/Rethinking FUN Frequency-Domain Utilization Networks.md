## 背景
- Leverage of  Frequency information in CNN
- DCT（离散余弦变换）![[Pasted image 20230321221529.png]]
## 贡献
- Frequency-domain Utilization Networks（FUN）：
	- a family of architectures that allow for trade-offs between model accuracy, model size, and inference latency
	- DCT-based, CNN architecture achieving state-of-the-art results on ImageNet classiﬁcation.
## 具体工作
- ResFUN Architecture：就是把RESNet的输入改成DCT的结果![[Pasted image 20230321221657.png]]
- eFUN（effective FUN）![[Pasted image 20230321221715.png]]
- 分析：DCT变化后，低频结果被High-Light，减少了网络输入的参数数量
- 这个工作太简单了，实验略过没看