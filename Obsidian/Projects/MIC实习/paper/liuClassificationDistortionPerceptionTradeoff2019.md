# On The Classification-Distortion-Perception Tradeoff
## Metadata
* Item Type: [[Conference paper]] 
* Authors: [[Dong Liu]], [[Haochen Zhang]], [[Zhiwei Xiong]] 
* Date: [[2019]] 
* Date Added: [[2023-03-29]] 
#* Tags: #unread 
## Zotero Links
* PDF Attachments
	- [Liu et al. - 2019 - On The Classification-Distortion-Perception Tradeo.pdf](zotero://open-pdf/library/items/8Z4NU757) 
* [Local library](zotero://select/items/1_NDI7LJDC) 
## Inroduction
### Background
- perceptual-distortion tradeoff
### Drawbacks
- **semantic quality** of the signal is also worthy consideration
	- i.e. the utility of the signal for recognition purpose
### Contribution
- extend the perceptual-distortion tradeoff to **Classification-Distortion-Perception(CDP) tradeoff**
	- measuring classification(semantic) quality by the **error rate achieved on the restored singal using a pre-trainde classifier network**
## Porposed Method
- problem formulation: restoration task(same as perceptual-distortion tradeoff)![[Pasted image 20230405205742.png]]
	- probablity mass function of X: $P_X(x)$
	- degradation model: $P_{Y|X}$ denoted by $p(y|x)$
	- restoration model: $P_{\hat{X}|Y}$ denoted by $p(\hat{x}|y)$
- porposed semantic quality: (represented by **the Classification Error** Rate by classificate the image into two classes)
	- there are two priori probabilities:$$P_X(x)=P_1P_{X_1}(x)+P_2P_{X_2}(x)$$$$P_Y(y)=P_1P_{Y_1}(y)+P_2P_{Y_2}(y)$$$$P_\hat{X}(\hat{x})=P_1P_{\hat{X}_1}(\hat{x})+P_2P_{\hat{X}_2}(\hat{x})$$
	- in two classes classification![[Pasted image 20230405211949.png]]
		- using a binary classifier![[Pasted image 20230405212036.png]]
	- Classification Error Rate![[Pasted image 20230405212051.png]]
- The CDP Tradeoff
	- Definition 1: CDP function, $c_0=c(.|R_0)$  is a pretrained binary classifier![[Pasted image 20230405212301.png]]
		- **minimize the error rate** of the classification of restored image, under the premise of **maintaining a constrains on perceptual quality and distortion**
	- Blind Restoration: the **distribution of the image to be resotred(Y) is unknown**
		- So it is impossible to achieve the optimal classifier for either retored image and degraded image, it is more practical to use a pretrained classifier
	- properties of CDP function![[Pasted image 20230405215347.png]]
		- The **CDP function is convex**:  when the error rate is already low, any improvement of classiﬁcation performance comes at the cost of higher distortion and worse perceptual quality.
## Experimental
- a toy example of CDP tradeoff![[Pasted image 20230405213543.png]] ![[Pasted image 20230405214243.png]]
- Example setting![[Pasted image 20230405214411.png]]
	- use the loss function below to showcase the tradeoff![[Pasted image 20230405214557.png]]
	- correspond result: EXP-1, EXP-2 and EXP-4 from top to bottom![[Pasted image 20230405214453.png]]
	- result of EXP-2 with different weights![[Pasted image 20230405214731.png]]
## Conclusions
- new point view: consider the restored signal quailty between the tradeoff between three metrics
	- **signal fidelity(distortion)**
	- **perceptual naturalness(quality)**
	- **semantic quality**
- what is represented by the error rate of the classifier?
	- so called semantic quality: represent some distributional properties of restored image
	- how can we formulate our representation of explaination of some kind of feature property? 
	- **Try to expand the formulation of our metric as soon!** 