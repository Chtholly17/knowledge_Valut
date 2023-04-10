# Why Are Deep Representations Good Perceptual Quality Features?
## Metadata
* Item Type: [[Conference paper]] 
* Authors: [[Taimoor Tariq]], [[Okan Tarhan Tursun]], [[Munchurl Kim]], [[Piotr Didyk]] 
* Date: [[2020]] 
* Date Added: [[2023-03-26]] 
# 
## Zotero Links
* PDF Attachments
	- [Full Text PDF](zotero://open-pdf/library/items/BCEG8L4U) 
* [Local library](zotero://select/items/1_4R6Y3YN4) 
## Inroduction
### Background
- perceptual loss: gain significant effect in representating of perceptual quality
- HVS: two fundamental properties
	- contrast sensitivity: human visual system is more sensitive to contrast than absolute luminance
		- CSF（Contrast Sensitive Function）：
			- a function of spatial frequency
			- observers have **lower contrast discrimination thresholds** at the spatial frequencies where the **CSF reaches a high value**
	- orientation selectivity
### Drawbacks
- there have been no systematic studies to determine the underlying reason.
	- why are some channels within a layer **perform better than the others?
	- can we select a **subset of channels** in a scale, to **eliminating the redundant feature channels that correlate poorly with human perception**
### Contribution
- capabilities of **pre-trained** perceptual loss: **the success in capturing basic human visual perception characteristics.**
- introduce two new formulations: **measuring the spatial frequency and orientation selectivity of the features channels** based on mean channel activations
## Porposed Method
- Input![[Pasted image 20230330133229.png]]
	- a series of sinusoidal gratings with different spatial frequency（gratings）
		- use symmetric gracting patterns as input to isolate the effect of spatial orientation
		- cal mean activation maps of each channel, as a function of spatial frequency of input signal
	- input a series of gratings pattern with different spatial orientation denoted by theta, with a fixed spatial frequency 
- Measurement of Spatial Frequency Sensitivity
	- Basic hypothesis: channels that exhibit **higher sensitivity to changes in the spatial frequency** are more likely to detect visual distortions
	- $\mu_1$ score: quantifies the average frequency sensitivity of a CNN channel![[Pasted image 20230330143729.png]]
		- k: denotes the convolution layer
		- m: denote the channel in a specific scale 
		- a_m^k: denotes the mean activation of a channel
		- f: spatial frequency in cycles(0-2pi) per degree
	- example: channel-1 has a higher frequency sensitivity($\mu_1$value) at the frequencies where CSF hit the peak![[Pasted image 20230330144029.png]]
		- which means that channel-1 have the desired behavior of responding to spatial frequency changes
		- mu_1 values of channels in RELU2_2 layer of the VGG-16(denoted in red)![[Pasted image 20230330145050.png]]
- Measurement of the Orientation Selectivity
	- For each channel, select a fix spatial frequency, which have the highest mu_1 score among 0-2pi
	- $\hat{a}^k_m$: represents the orientation with which the coresponding channel have the highest activation score![[Pasted image 20230330155109.png]]
		- theta represent a specific orientation
	- cal mu_2 score based on the score above![[Pasted image 20230330155236.png]]
		- theta: denote the orientation, varying from -pi/2 to pi/2 per degree
		- Orientation Selectivity of two channels![[Pasted image 20230330160404.png]]
		- mu_2 values of channels in RELU2_2 layer of the VGG-16(denoted in red)![[Pasted image 20230330160453.png]]
- Perceptual Efficacy(PE): k denots one layer of the CNN, whose output feature has m channles![[Pasted image 20230330160613.png]]
	- combine mu_1 and mu_2 score above
	- represents a overall goodness of a feature channel
## Experimental
- Setup
	- channel with high score![[Pasted image 20230330162328.png]]
	- channel with lower socre![[Pasted image 20230330162350.png]]
- test on IQA![[Pasted image 20230330162458.png]]
- Visual Evalation of features: reconstruct image from feature with high and low PE score respectively![[Pasted image 20230330162634.png]]
	- H represent a structure or semantic information(more visually relevant)
	- L represent a texture information(color is encoded in the channels with lower PE scores)
- test on Super Resolution![[Pasted image 20230330162803.png]]
## Conclusions
- Porposed PE score can be a measurement of our metric: 
	- Cal PE of the original channel and the output of our metric respectively
- Can we explain our metric in bais of **Spatial Frequency Sensitivity** and **Orientation Selectivity**
	- Is our metric improve mu_1 score or mu_2 score? We can use explaination in this artical