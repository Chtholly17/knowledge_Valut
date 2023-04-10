# Unsupervised Image Denoising with Frequency Domain Knowledge
## Metadata
 
* Authors: [[Nahyun Kim]], [[Donggon Jang]], [[Sunhyeok Lee]], [[Bomi Kim]], [[Dae-Shik Kim]] 
* Date: [[2021-11-29]] 
* Date Added: [[2023-03-26]] 
#* Tags: #unread 
## Zotero Links
* PDF Attachments
	- [Kim et al. - 2021 - Unsupervised Image Denoising with Frequency Domain.pdf](zotero://open-pdf/library/items/NBGPFX35) 
* [Local library](zotero://select/items/1_8TX38RMA) 
## Inroduction
### Background
- Image Denoising
### Drawbacks
- most of learning based Image Denoising method: 
	- utilize only one-sided information from the spatial domain
	- without considering frequency domain information.
### Contribution
- a GAN-based frequency-sensitive unsupervised denoising method.
### Motivation
- Image Denoising: noisy lies in the high-frequency bands![[Pasted image 20230329120652.png]]
- Fourier domain semantics: semantic information lies in the low-frequency bands. Furthermore

## Porposed Method
- Framework: ![[Pasted image 20230329121935.png]]
	- The input noise image x_n and clean image y_c is unpaired
	- Loss for G_{n2c} : 
		- Loss for G_{n2c} and D_C(Discriminator)Pasted image 20230329123908.png]]
		- Loss for G_{n2c} and D_T(Discriminator for texture)![[Pasted image 20230329123609.png]]
			- M_{shift} is a random color algorithm. it's adopted here t0 maintain texture consistancy
		- Loss for G_{n2c} and D_S(Discriminator for Spectral)![[Pasted image 20230329124312.png]]
	- Loss to impose cycle consistancy(for G_{c2n})
		- L_{Recon}: **impose cycle consistancy in Fourier domain and Spatial domain respectivly![[Pasted image 20230329122748.png]]
			- Frequency Reconstruction Loss: cycle consistency in Fourier Domain![[Pasted image 20230329121720.png]]
				- G_{n2c}: Generator which takes noised image as input and generates clean image
				- G_{c2n}: Adverse to Generator above to impose cycle consistancy
		- L_{CC} : impose cycle consistancy only in pixel domain using L1 distance![[Pasted image 20230329122835.png]]
	- Loss to perserve information in restored image
		- L_{VGG}: use perceptual loss to **ensure that extracted features from the noisy and denoised image are semantically invariant.![[Pasted image 20230329123035.png]]
			- 这里认为Lpips的意义在于减少semantic distance
		- L_{BG}: background loss, preserve background consistency between the noisy and denoised image.![[Pasted image 20230329123445.png]]
			- add blur to noise image and restored image respectively
			- cal the L1 distance between them
	- L_{TV} : Total Variance loss, impose the local smoothness and mitigate the artifacts in the restored image![[Pasted image 20230329123212.png]]
		- ▽w and ▽h are the operations to compute the gradients in terms of horizontal and vertical directions, 
  
## Experimental
- 定性结果：![[Pasted image 20230329124908.png]]
## Conclusions
- 将SFI使用在cycle GAN中
- 关于频域和空域的解释与之前类似，噪声主要存在于高频信息中
- 关于texture information preservation 的新解释：使用random color shift之后的结果送进评价器，说明某种意义上随机颜色位移后的结果代表了texture information
-  **Loss in Unsupervised Learning: data is unpaired, we need to calculate the distribution between different domain instead of distance of a single image and its corresponding ground truth**