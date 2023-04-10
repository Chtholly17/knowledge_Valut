# Kernel-aware Dual-level Statistical Distance for Image Quality Assessment with Relaxed Reference
## Metadata
* Item Type: [[Article]] 
* Authors: [[Xinpeng Li]], [[Ting Jiang]], [[Qingbo Wu]], [[Hongliang Li]], [[Bing Zeng]], [[Shuaicheng Liu]] 
* Date Added: [[2023-03-27]] 
#unread 
## Zotero Links
* PDF Attachments
	- [Li et al. - Kernel-aware Dual-level Statistical Distance for I.pdf](zotero://open-pdf/library/items/I6LXGLGL) 
* [Local library](zotero://select/items/1_7S8P3XQS) 
## Inroduction
### Background
- NAR-IQA(non-aligned reference IQA)
	- content similar
	- content variant
- deep IQA methods
## Motivation

- non-aligned reference approaches, larger receptive fields are preferred.
- spectral convolution theorem: **change a value in frequency domain would have a global impact on the original signal**
- measurement of distance: using MMD(Distribution Distance similar to SWD) instead of position-related distance

### Drawbacks

- Current NAR-IQA method doesn't leverage the larger receptive with frequent domain

### Contribution

- KDSD：Kernel-aware Dual-level Statistical Distance

## Porposed Method

- Framework：![[Pasted image 20230328114108.png]]
- MDFE module![[Pasted image 20230328220415.png]]
	- Global Branch（上面一半）：由spectral convolution theorem，这部分对频域进行了操作，因此包含了Global 信息
		- 首先进行FFT![[Pasted image 20230328220709.png]]
		- 然后实部虚部分离，concat在一起![[Pasted image 20230328220733.png]]
		- 卷积+Batch Normal+激活
		- 从实数转化回虚数![[Pasted image 20230328220910.png]]
		- IFFT
	- Local 和 Local 、Global 和 Global feature之间计算MMD，分别为Dl和Dg
- 然后分别用两个MLP，f和g，投射到两个向量，然后用一个网络投射成最后的分数，计算最后和ground truth分数之间的差别
## Experimental
- 定量结果：![[Pasted image 20230329003901.png]]
## Conclusions
- 这篇文章用到的思路和我们有些相似，区别在于
	- MDFE module中使用了卷积，具有可学习参数
	- 输出的MMD计算结果，分别使用了MLP进行回归，这个过程中应该进行了fine-tune，一定是对domain overfitting的，我们的方法是close-form的，可以具有更加通用的应用