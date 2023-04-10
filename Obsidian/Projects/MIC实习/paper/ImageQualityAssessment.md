# Image Quality Assessment: Measuring Perceptual Degradation via Distribution Measures in a Deep Feature Space
## Metadata
* Item Type: [[Conference paper]] 
* Date Added: [[2023-03-29]] 
# 
## Zotero Links
* PDF Attachments
	- [f746ad1b860717ac3620b76979ba712d.pdf](zotero://open-pdf/library/items/VTKFEYZD) 
* [Local library](zotero://select/items/1_XLFBWVLL) 
## Inroduction
### Background
- FR-IQA
- perceptual metric
	- pixel-by-pixel does not lead to a perceptual 
 - deep features of the Visual Geometry Group 16 (VGG16) network contain structure and texture information
### Drawbacks
- pixelwise comparison process often ignores the pixel correlations among deep features
- using an SSIM variant to compare deep features is not a robust method for some distortion samples![[Pasted image 20230330201841.png]]
### Contribution
- A deep-based FR-IQA measures
	- view deep features as statistical distributions
	- introduce distribution measures to address the pixel correlation issue.
	- this measures can be applied in some image reconstruction tasks to obtain more visually satisfactory results
### Motivation
- **the HVS pursues perceptual information fidelity** when receiving two stimuli. 
	- pixelwise fidelity measures such as the MSE fail to serve as perceptual measures.
- **we regard deep features as a type of distribution** and then use distribution measures to estimate **pixel correlation fidelity.**
## Porposed Method
- overall philosophy![[Pasted image 20230330203451.png]]
- framework![[Pasted image 20230330204007.png]]
	- Deep-DistributionDisances![[Pasted image 20230330204112.png]]
	- a weighted Euclidean norm for each dstribution measure:![[Pasted image 20230330204259.png]] ![[Pasted image 20230330204550.png]]
## Experimental
- IQA![[Pasted image 20230330205009.png]]
- 定量结果![[Pasted image 20230330205023.png]]
## Conclusions
- motivation of this artical
	- properties of HVS:  perceptual information fidelity **which is very different from pixelwise**