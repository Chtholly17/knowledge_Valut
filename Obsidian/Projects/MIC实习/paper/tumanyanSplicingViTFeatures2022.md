# Splicing ViT Features for Semantic Appearance Transfer
## Metadata
* Item Type: [[Conference paper]] 
* Authors: [[Narek Tumanyan]], [[Omer Bar-Tal]], [[Shai Bagon]], [[Tali Dekel]] 
* Date: [[2022]] 
* Date Added: [[2023-03-28]] 
#* Tags: #unread 
## Zotero Links
* PDF Attachments
	- [Full Text PDF](zotero://open-pdf/library/items/X7VX38XF) 
* [Local library](zotero://select/items/1_6C5C277P) 
## Inroduction
### Background
- style-transfer
	- input two images: apperance image and structure image
	- output a image with structure and appreance from two input images respectively
### Contribution
- a style-transfer method:
	- achieve high quality results in vairety of in-the-wild image pairs
	- does not require adversrial learning and additional information as input(such as semantic segmentation)
- DINO-ViT: a Vision Transformer model pre-trained in a self-supervised manner
	- a novel **representation of apperance**(texture, style and so on) **and structure**(semantic) **information** in DINO-ViT feature space
	- DINO-ViT: a transformer, **is also a feature extractor as VGG**
## Porposed Method
- framework![[Pasted image 20230404220919.png]]
	- match structure and appearnce of output image with two input image respectively
- 
## Experimental
## Conclusions