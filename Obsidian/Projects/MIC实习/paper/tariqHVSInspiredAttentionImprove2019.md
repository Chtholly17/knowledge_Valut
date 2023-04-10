# A HVS-Inspired Attention to Improve Loss Metrics for CNN-Based Perception-Oriented Super-Resolution
## Metadata
* Item Type: [[Conference paper]] 
* Authors: [[Taimoor Tariq]], [[Juan Luis Gonzalez Bello]], [[Munchurl Kim]] 
* Date: [[2019]] 
* Date Added: [[2023-03-27]] 
# 
## Zotero Links
* PDF Attachments
	- [Tariq et al. - 2019 - A HVS-Inspired Attention to Improve Loss Metrics f.pdf](zotero://open-pdf/library/items/H8VR2N2A) 
* [Local library](zotero://select/items/1_PM3BK8TQ) 
## Inroduction
### Background
- perceptual loss
- HVS: contrast sensitivty and spatial frequency
- Singal Image Super Resolution(SISR)
### Contribution
- the author designed a **spatial attention map** identifying **which region of feature**(extracted by VGG in perceptual loss) are **more perceptual 
### Motivation
- there can be an inﬁnite number of of possible HRs that **have the same low-frequency components but different high frequency components** 
## Porposed Method
- framework![[Pasted image 20230403220903.png]]
- Map Generating： use featuure map of GT to generate a CSF based map![Pasted image 20230404140108.png]]
	- Convert the image to Frequency domain![[Pasted image 20230404135526.png]]
	- cal the spatial frequency of a point in frequency domain![[Pasted image 20230404135615.png]] ![[Pasted image 20230404135624.png]]
	- use the narrow band of spatial frequnecies around the peak of CSF![[Pasted image 20230404135727.png]]
	- convert it back to spatial domain![[Pasted image 20230404135743.png]]
	- normalization![[Pasted image 20230404135753.png]]
- Spatially Attentive (perceptual) Loss Function: ![[Pasted image 20230404140723.png]]
## Experimental
- Quantitative experiment
	- Perceptual-Distotion tradeoff![[Pasted image 20230404141700.png]]
		- perceptual quality measurement: Perceptual Index(PI)![[Pasted image 20230404141342.png]]
		- Distortion is measured by SSIM and PSNR
		- Effectiveness of technique: deliver a better trade-off i.e lesser PI at the same SSIM.
- Qualitative experiment: 
	- x4 Super Resolution![[Pasted image 20230404141829.png]]
## Conclusions
- method based on CSF: might not be suitable for our framework
	- we do not emphasis on some  peculiar frequency compoments
	- Its better to explain our method in point of **perceputual quality and distortion**
- this artical provide us a method to **cal the spatial frequency representation of a point in Frouier domain**