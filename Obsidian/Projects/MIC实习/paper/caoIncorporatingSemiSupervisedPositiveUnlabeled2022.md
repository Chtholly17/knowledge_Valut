# Incorporating Semi-Supervised and Positive-Unlabeled Learning for Boosting Full Reference Image Quality Assessment
## Metadata
* Item Type: [[Conference paper]] 
* Authors: [[Yue Cao]], [[Zhaolin Wan]], [[Dongwei Ren]], [[Zifei Yan]], [[Wangmeng Zuo]] 
* Date: [[2022]] 
* Date Added: [[2023-03-27]] 
## Zotero Links
* PDF Attachments
	- [Cao et al. - 2022 - Incorporating Semi-Supervised and Positive-Unlabel.pdf](zotero://open-pdf/library/items/N8692B3M)
* [Local library](zotero://select/items/1_B53SVRMR) 
## Inroduction
### Background
- FR IQA
- Semi-Supervies IQA method(PU learninga)
### Drawbacks
- Difficluty to acssess labeled image set
### Contribution
- FR-IQA: deploying **spatial attention** in computing difference map of Distorted and Reference Image
## Porposed Method
- FR-IQA Network
	- framework![[Screenshot 2023-04-04 at 15.04.55.png]]
		- replace the max pooling in VGG with L_2 pooling(to avoid aliasing)
		- Dual Attention Blocks(DAB): contian spatial Attention module and channel Attention module
		- Local SW Distance: for features of Distorted image and Reference image in a scale![[Pasted image 20230404202352.png]]
			- devide feature in spatial domain into patches($R^{C*p*p}$)
			- for patches with same spatial location from distorted and reference feature, project the patch for m($m=C/2$)times
			- cal the 1-D WD between the same projections
			- the total result betweem two patches is a vector($R^m$)
			- thus the localSW Distance of the distorted and reference image in one scale is $R^{\frac{H}{p},\frac{W}{p},m}$
		- spatial attention block (**Obviously, the contribution of image region to visual quality is spatially varying.**)![[Pasted image 20230404203946.png]]
			- here use the GT(reference) to generate the **Spatial Attention map**
## Experimental
- Quantitivate Experiment
	- IQA task: ![[Pasted image 20230404204438.png]]
## Conclusions
- This paper and previous paper[[tariqHVSInspiredAttentionImprove2019]] both leverage the GT to generate the spatial Attention map. Indicating that it is a **common approach in Attention map generating**