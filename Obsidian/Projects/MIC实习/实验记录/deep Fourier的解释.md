- 傅立叶变化产生的Phase图像描绘了原图的structure information（semantic information）[[Spatial-Frequency Domain Information Integration for Pan-Sharpening]]
- Amplitude（幅值）描绘了原图中的style、光照（lighting）、纹理（texture）等信息
- 傅立叶变化的结果：中间和四周分别代表低频信息和高频信息
## pan- sharping
- learning in frequency information allows the image-wide receptive ﬁeld that models the global contextual information[[Frigo, M., Johnson, S.G.: FFTW: an adaptive software architecture for the FFT.]]
	- boosting the information representation and model capability.
- direct emphasis on the frequency content is capable of better reconstructing the global information, thus improving the pan-sharpening performance
- the frequency branch generates the global information representation![[Pasted image 20230320204109.png]]
## t-SNE
- 关于频域幅值和相位信息的解释，可以使用t-SNE[[Van der Maaten, L., Hinton, G.: Visualizing data using t-SNE

## Denoising
- [[Kim et al. - 2021 - Unsupervised Image Denoising with Frequency Domain.pdf]]
- 1D representation of the Fourier power spectrum is sufﬁcient to highlight spectral differences